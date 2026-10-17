Current implementation in the ebook plugin to get html for remote pdf and epub generation with `onExportHtmlLoaded` integration:

```php
use Typemill\Events\OnExportHtmlLoaded;
```

# 1. PDF generation

For the remote PDF generation, the whole book is rendered as a single HTML document by Twig and then sent to the WeasyPrint service.
Before the call, dispatch `onExportHtmlLoaded` so plugins like PlantUML, Mermaid or Math can replace dynamic markup with static images.

```php
	public function createRemotePdf(Request $request, Response $response, $args)
	{
		$params 		= $request->getParsedBody();
		$settings 		= $this->getSettings();
		$auth_url 		= $settings['auth_url'] ?? 'https://service.typemill.net';
		$kixote_url 	= $settings['kixote_url'] ?? 'https://kixote.typemill.net';

		if($auth_url == 'https://service.typemill.net')
		{
			$license 		= new License();
			$token 			= $license->getToken();

			if(!$token)
			{
				$response->getBody()->write(json_encode([
					'message' 	=> $license->getMessage(),
				]));

				return $response->withHeader('Content-Type', 'application/json')->withStatus(422);			
			}
		}
		else
		{
			# individual authenticator, how to handle this? Just an url where you return a token, right? 
			# So it would be the kixote app with the authentication login and an api key
		}

		/* Get the book data */

		$projectname 	= $params['projectname'] ?? false;
		$itempath 		= $params['itempath'] ?? false;
		$name 			= $params['name'] ?? false;
		$baseurl 		= $this->urlinfo['baseurl'];
		$dispatcher 	= $this->getDispatcher();
		$settings 		= $this->getSettings();
		$pathToContent	= $settings['rootPath'] . DIRECTORY_SEPARATOR . 'content';
		$storage 		= new StorageWrapper($settings['storage']);

		$ebookdata 		= $this->getEbookDataForPdf($projectname, $itempath, $storage);
		if(!$ebookdata)
		{
			$response->getBody()->write(json_encode([
				'message' 	=> $this->ebookerror
			]));

			return $response->withHeader('Content-Type', 'application/json')->withStatus(422);			
		}

		$navigation 	= $this->getNavigationForPdf($projectname, $itempath, $ebookdata, $storage);
		if(!$navigation)
		{
			$response->getBody()->write(json_encode([
				'message' 	=> $this->ebookerror
			]));

			return $response->withHeader('Content-Type', 'application/json')->withStatus(422);
		}

		$parsedown 		= $this->configureParsedown($baseurl, $dispatcher, $ebookdata, );

		# activate sectionNumbers (headline numbers)
		$sectionNumbers = $this->initiateSectionNumbers($ebookdata);

		$bookcontent 	= $this->generateContent([], $navigation, $pathToContent, $parsedown, $ebookdata, $chapterlevel = NULL, $sectionNumbers);
		$toc = false;
		if(isset($bookcontent['toc']) && !empty($bookcontent['toc']))
		{
			$toc = $bookcontent['toc'];
			unset($bookcontent['toc']);
		}

		$bookcontent 	= $this->transformRelativeLinks($bookcontent, $toc, $baseurl);

		# add the thumb index:
		$thumbindex 	= $this->generateThumbIndex($bookcontent);

		# we have to dispatch onTwigLoaded to get javascript from other plugins
		$dispatcher->dispatch(new OnTwigLoaded(false), 'onTwigLoaded');

		$twig   		= $this->getTwig();
		$loader 		= $twig->getLoader();
		$loader->addPath(__DIR__ . '/templates', 'ebooks');
		$loader->addPath(__DIR__ . '/booklayouts/' . $ebookdata['layout'], 'booklayouts');
	
		$booklayouts 	= $this->scanEbooklayouts();

		# load customcss
		$customcss = $storage->checkFile('cacheFolder', '', 'ebooklayout-' . $ebookdata['layout'] . '-custom.css');
		if($customcss)
		{
			$this->container->get('assets')->addCSS($baseurl . '/cache/ebooklayout-' . $ebookdata['layout'] . '-custom.css');
		}

		$data = [
			'settings' 		=> $settings, 
			'base_url' 		=> $baseurl, 			
			'ebookdata' 	=> $ebookdata, 
			'booklayout'	=> $booklayouts[$ebookdata['layout']],
			'book' 			=> $bookcontent,
			'toc' 			=> $toc,
			'thumbindex'	=> $thumbindex
		];

		$html = $twig->fetch('@booklayouts/index.twig', $data);

		# allow plugins (PlantUML, Mermaid, Math...) to generate static assets
		$html = $this->prepareExportHtml($html, [], $baseurl);


		/* CALL TO KIXOTE WEASYPRINT */
		
		$api = new ApiCalls();
		$api->setTimeout(60);

		if($kixote_url == 'https://kixote.typemill.net')
		{
			# let us check if we can authenticate with the token 
 #			$apiResponse = $api->makeGetCall($kixote_url . '/check/', $authHeader = 'Authorization: Bearer ' . $token);
 #			$apiResponse = $api->makeGetCall($kixote_url . '/check/auth', $authHeader = 'Authorization: Bearer ' . $token);
 #			$apiResponse = $api->makeGetCall($kixote_url . '/check/limit', $authHeader = 'Authorization: Bearer ' . $token);
 #			$apiResponse = $api->makeGetCall($kixote_url . '/check/rate', $authHeader = 'Authorization: Bearer ' . $token);

			$metatitle 		= $name . '_' . date('Ymd-Hi');

			$bookdata = [
			  "html" 					=> $html,
 #			  "css" 					=> "h1 { color: red; }",     	// string or list of strings
			  "metadata" 				=> ["title" => $metatitle],  		// dictionary
			  "zoom" 					=> 1.0,                    		// number
			  "base_url" 				=> $baseurl,					// string
			  "presentational_hints" 	=> true,               			// boolean
			  "optimize_size" 			=> ["images", "fonts"],			// list
			  "filename" 				=> $name . '.pdf'          // string
			];

			$apiResponse = $api->makePostCall(
				$kixote_url . '/book/pdf', 
				$bookdata,
				$authHeader = 'Authorization: Bearer ' . $token
			);
			
			if(!$apiResponse)
			{
				$response->getBody()->write(json_encode([
					'message' 	=> "call to Kixote server failed",
					'data'		=> $api->getError()
				]));

				return $response->withHeader('Content-Type', 'application/json')->withStatus(422);
			}

			$storage 	= new StorageWrapper($settings['storage']);
			$filefolder = $storage->getFolderPath('fileFolder');
			$filename 	= $metatitle . '.pdf';
			$filepath 	= $filefolder . DIRECTORY_SEPARATOR . $filename;

			file_put_contents($filepath, $apiResponse);

			$tokenstats = $api->makeGetCall($kixote_url . '/check/rate', $authHeader = 'Authorization: Bearer ' . $token);

			// build URL for the frontend
			$fileUrl = '/media/files/' . $filename;

			$response->getBody()->write(json_encode([
			    'message' 		=> 'PDF successfully generated',
			    'url'     		=> $fileUrl,
			    'tokenstats'	=> $tokenstats
			]));

			return $response->withHeader('Content-Type', 'application/json');
		}

		$response->getBody()->write(json_encode([
			'message' 	=> 'Something went wrong',
		]));

		return $response->withHeader('Content-Type', 'application/json')->withStatus(422);
	}
```

# 2. EPUB generation

For EPUB generation, each chapter is added separately. Therefore dispatch `onExportHtmlLoaded` per-chapter, passing the page item so plugins can resolve page-relative links.

```php
	# generates and returns the epub file
	public function createEpub(Request $request, Response $response, $args)
	{
		if(ob_get_level() > 0)
		{
			ob_end_flush();
		}

		ini_set('error_reporting', E_ALL);
		ini_set('display_errors', '0');

		$projectname 	= $request->getQueryParams()['projectname'] ?? false;
		$itempath 		= $request->getQueryParams()['itempath'] ?? false;
		$baseurl 		= $this->urlinfo['baseurl'];
		$dispatcher 	= $this->getDispatcher();
		$settings 		= $this->getSettings();
		$pathToContent	= $settings['rootPath'] . DIRECTORY_SEPARATOR . 'content';
		$storage 		= new StorageWrapper($settings['storage']);

		$ebookdata 		= $this->getEbookDataForPdf($projectname, $itempath, $storage);
		if(!$ebookdata)
		{
			$response->getBody()->write(json_encode([
				'message' 	=> $this->ebookerror
			]));

			return $response->withHeader('Content-Type', 'application/json')->withStatus(422);			
		}

		$navigation 	= $this->getNavigationForPdf($projectname, $itempath, $ebookdata, $storage);
		if(!$navigation)
		{
			$response->getBody()->write(json_encode([
				'message' 	=> $this->ebookerror
			]));

			return $response->withHeader('Content-Type', 'application/json')->withStatus(422);
		}

		# setup parsedown with individual settings
		$parsedown = new ParsedownExtension($baseurl);

		$parsedown->withSpanFootnotes();

		# check if shortcodes should be rendered
		if(isset($ebookdata['disableshortcodes']) && $ebookdata['disableshortcodes'])
		{
			# empty array will stop all shortcodes
			$parsedown->setAllowedShortcodes(array());
		}
		elseif(isset($ebookdata['activeshortcodes']) && is_array($ebookdata['activeshortcodes']) && !empty($ebookdata['activeshortcodes']))
		{
			# only selected shortcodes will be rendered
			$parsedown->setAllowedShortcodes($ebookdata['activeshortcodes']);
		}

		$bookcontent 	= $this->generateContent([], $navigation, $pathToContent, $parsedown, $ebookdata);
		unset($bookcontent['toc']);

		##############
		# START EPUB #
		##############

		# setting timezone for time functions used for logging to work properly
		date_default_timezone_set('Europe/Berlin');
		$log = new Logger("eBook Log", TRUE);

		$fileDir = '.';
		$fileDir = $settings['rootPath'];
		
		# ePub 3 is not fully implemented. but aspects of it is, in order to help implementers.
		# ePub 3 uses HTML5, formatted strictly as if it was XHTML but still using just the HTML5 doctype (aka XHTML5)
		$book = new EPub(EPub::BOOK_VERSION_EPUB3, "en", EPub::DIRECTION_LEFT_TO_RIGHT); // Default is ePub 2
		$log->logLine("new EPub()");
		$log->logLine("EPub class version.: " . EPub::VERSION);
		$log->logLine("Zip version........: " . Zip::VERSION);
		$log->logLine("getCurrentServerURL: " . URLHelper::getCurrentServerURL());
		$log->logLine("getCurrentPageURL..: " . URLHelper::getCurrentPageURL());

		if(!isset($ebookdata['epubtitle']) OR $ebookdata['epubtitle'] == '')
		{
			$response->getBody()->write('The ePub title is missing.');
			
			return $response->withStatus(422);
		}

		# Title and Identifier are mandatory!
		$book->setTitle($ebookdata['epubtitle']);
		
		# Could also be the ISBN number, preferred for published books, or a UUID like https://www.php.net/manual/en/function.uniqid.php
		if(isset($ebookdata['epubidentifierisbn']) && $ebookdata['epubidentifierisbn'] != '')
		{
			$book->setIdentifier($ebookdata['epubidentifierisbn'], EPub::IDENTIFIER_ISBN);
		}
		elseif(isset($ebookdata['epubidentifieruuid']) && $ebookdata['epubidentifieruuid'] != '')
		{
			$book->setIdentifier($ebookdata['epubidentifieruuid'], EPub::IDENTIFIER_UUID);
		}
		elseif(isset($ebookdata['epubidentifieruri']) && $ebookdata['epubidentifieruri'] != '')
		{
			$book->setIdentifier($ebookdata['epubidentifieruri'], EPub::IDENTIFIER_URI);
		}
		else
		{
			$response->getBody()->write('There is no epub identifier. An identifier is mandatory, we cannot create an ePub without that.');
			return $response;
		}

		# Not needed, but included for the example, Language is mandatory, but EPub defaults to "en". Use RFC3066 Language codes, such as "en", "da", "fr" etc.
		if(isset($settings['langattr']) && $settings['langattr'] != '')
		{
			$book->setLanguage($settings['langattr']);
		}

		if(isset($ebookdata['epubdescription']) && $ebookdata['epubdescription'] != '')
		{
			$book->setDescription($ebookdata['epubdescription']);
		}

		if(isset($ebookdata['epubauthorfirstname']) && $ebookdata['epubauthorfirstname'] != '' && isset($ebookdata['epubauthorlastname']) && $ebookdata['epubauthorlastname'] != '')
		{
			$book->setAuthor($ebookdata['epubauthorfirstname'] . ' ' . $ebookdata['epubauthorlasttname'], $ebookdata['epubauthorlasttname'] . ', ' . $ebookdata['epubauthorfirstname']);
		}

		if(isset($ebookdata['epubpublishername']) && $ebookdata['epubpublishername'] != '')
		{
			$publisherurl =  isset($ebookdata['epubpublisherurl']) ? $ebookdata['epubpublisherurl'] : '';
			$book->setPublisher($ebookdata['epubpublishername'], $publisherurl);
		}

		# Strictly not needed as the book date defaults to time().
		$book->setDate(time());

		# As this is generated, this _could_ contain the name or licence information of the user who purchased the book, if needed. If this is used that way, the identifier must also be made unique for the book.
		if(isset($ebookdata['epubrights']) && $ebookdata['epubrights'] != '')
		{
			$book->setRights($ebookdata['epubrights']);
		}

		if(isset($ebookdata['epubsourceurl']) && $ebookdata['epubsourceurl'] != '')
		{
			$book->setSourceURL($ebookdata['epubsourceurl']);
		}

		$book->addDublinCoreMetadata(DublinCore::CONTRIBUTOR, "PHP");

		if(isset($ebookdata['epubsubjects']) && $ebookdata['epubsubjects'] != '' )
		{
			$subjects = array_map('trim', explode(',', $ebookdata['epubsubjects']));

			foreach($subjects as $subject)
			{
				$book->setSubject($subject);
			}
		}

		# Insert custom meta data to the book, in this case, Calibre series index information.
		CalibreHelper::setCalibreMetadata($book, $ebookdata['epubtitle'], "3");

		$log->logLine("Set up parameters");

		# ePub 3 uses a variant of HTML5 called XHTML5
		$content_start =
			"<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n"
			. "<html xmlns=\"http://www.w3.org/1999/xhtml\" xmlns:epub=\"http://www.idpf.org/2007/ops\">\n"
			. "\t<head>"
			. "\t\t<meta http-equiv=\"Default-Style\" content=\"text/html; charset=utf-8\" />\n"
			. $book->getViewportMetaLine() // generate the viewport meta line if the viewport is set.
			. "\t\t<link rel=\"stylesheet\" type=\"text/css\" href=\"styles.css\" />\n"
			. "\t\t<title>" . $ebookdata['title'] . "</title>\n"
			. "\t</head>\n"
			. "\t<body>\n";

		$bookEnd = "\t</body>\n</html>\n";

		$log->logLine("Add css");

		$cssData = '';
		$cssPath = $settings['rootPath'] . 'plugins' . DIRECTORY_SEPARATOR . 'ebooks' . DIRECTORY_SEPARATOR . 'booklayouts' . DIRECTORY_SEPARATOR . $ebookdata['layout'] . DIRECTORY_SEPARATOR . 'epub.css';
		if(file_exists($cssPath))
		{
			$log->logLine("Custom CSS found");

			if(!is_readable($cssPath))
			{
				$log->logLine("Custom CSS not readable");
			}
			else
			{
				$cssData = file_get_contents($cssPath);
			}
		}
		else
		{
			# add standard epub css
			$cssPath = $settings['rootPath'] . 'plugins' . DIRECTORY_SEPARATOR . 'ebooks' . DIRECTORY_SEPARATOR . 'epubcss' . DIRECTORY_SEPARATOR . 'epub-starter-kit.css';		
			$cssData = file_get_contents($cssPath);
		}

		$book->addCSSFile("styles.css", "css1", $cssData);

		# A better way is to let EPub handle the image itself, as it may need resizing. Most e-books are only about 600x800
		# pixels, adding mega-pixel images is a waste of place and spends bandwidth. setCoverImage can resize the image.
		# When using this method, the given image path must be the absolute path from the servers Document root.

		if(isset($ebookdata['epubcover']) && file_exists($settings['rootPath'] . $ebookdata['epubcover']) )
		{
			$original = str_replace('/media/live/', '/media/original/', $ebookdata['epubcover']);
			$book->setCoverImage($settings['rootPath'] . $original);
		}

		# setCoverImage can only be called once per book, but can be called at any point in the book creation.
		$log->logLine("Set Cover Image");

		# Create the cover
		$cover = $content_start . "<h1>" . $ebookdata['epubtitle'] . "</h1>\n";
		if(isset($ebookdata['epubsubtitle']) && $ebookdata['epubsubtitle'] != '')
		{ 	
			$cover .= "<h2>" . $ebookdata['epubsubtitle'] . "</h2>\n";
		}
		if(isset($ebookdata['epubauthor']) && $ebookdata['epubauthor'] != '')
		{
			$cover .= "<p>" . $ebookdata['epubauthor'] . "</p>\n"; 
		}
		$cover .= $bookEnd;

		$tocName = (isset($ebookdata['epubtocname']) && $ebookdata['epubtocname'] != '' ) ? $ebookdata['epubtocname'] : 'Table of Contents';
		$book->addChapter($tocName, "TOC.xhtml", NULL, false, EPub::EXTERNAL_REF_IGNORE);
		$log->logLine("ToC added");

		$titleName = (isset($ebookdata['epubtitlepage']) && $ebookdata['epubtitlepage'] != '' ) ? $ebookdata['epubtitlepage'] : 'Notices';
		$book->addChapter($titleName, "Cover.xhtml", $cover);
		$log->logLine("Cover added");

		if(isset($ebookdata['epubimprint']) && $ebookdata['epubimprint'] != '')
		{
			$imprinttitle  	= (isset($ebookdata['epubimprinttitle']) && $ebookdata['epubimprinttitle'] != '') ? $ebookdata['epubimprinttitle'] : 'Imprint';
			$imprintarray 	= $parsedown->text($ebookdata['epubimprint'], $itemUrl = false);
			$imprinthtml 	= $parsedown->markup($imprintarray, $itemUrl = false);

			# allow plugins (PlantUML, Mermaid, Math...) to generate static assets
			$imprinthtml = $this->prepareExportHtml($imprinthtml, [], $baseurl);

			$imprint = $content_start;
			$imprint .= "<h1>" . $imprinttitle . "</h1>";
			$imprint .= $imprinthtml;
			$imprint .= $bookEnd;

			$book->addChapter($imprinttitle, "Imprint.xhtml", $imprint);

			# create the content here
			$log->logLine("Imprint added");
		}

		# create the content here
		$log->logLine("Build Chapters");

		$ChapterName 	= (isset($ebookdata['epubchaptername']) && $ebookdata['epubchaptername'] != '' ) ? $ebookdata['epubchaptername'] . ' ' : '';
		$prefix 		= '';
		$prefixNumber	= (isset($ebookdata['epubchapternumber']) && $ebookdata['epubchapternumber']) ? true : false;
		$chapNumArray 	= ['0' => 1]; # initial chapter
		$lastLevel 		= false;

		foreach($bookcontent as $chapter)
		{
			if($lastLevel)
			{
				# we have to reduce by 1 so we find the correct key in chapNumArray starting with 0
				$chapterKey = $chapter['level']-1;
				
				# if we are in the same level
				if( ($chapter['level'] == $lastLevel) ) 
				{
					if(!isset($chapNumArray[$chapterKey]))
					{
						print_r($chapNumArray);
						die('the chapnum level . ' . $chapter['level'] . ' does not exist.' );
					}

					# increment
					$chapNumArray[$chapterKey]++;
				}
				# if we went one level deeper in the hierarchy
				elseif($chapter['level'] > $lastLevel)
				{
					if(isset($chapNumArray[$chapterKey]))
					{
						print_r($chapNumArray);
						die('the chapnum level . ' . $chapter['level'] . ' already exists.' );
					}

					# we initialize the deeper level with 1
					$chapNumArray[$chapterKey] = 1;

					# that means we add a sub-level
					$book->subLevel();
				}
				# if we went one level up in the hierarchy
				elseif($chapter['level'] < $lastLevel)
				{
					# we cut/shorten the chapNumArray accordingly...
					$chapNumArray = array_slice($chapNumArray, 0, $chapter['level']);

					# and we increment the current chapter level
					$chapNumArray[$chapterKey]++;

					# and we adjust the level for the ePub accordingly
					$book->setCurrentLevel($chapter['level']);
				}
			}

			$lastLevel = $chapter['level'];
			
			$readableChapNum = implode(".", $chapNumArray);

			$filename = "Chapter00";

			if($chapNumArray[1] > 9)
			{
				$filename = "Chapter0";
				if($chapNumArray[1] > 99)
				{
					$filename = "Chapter";
				}
			}
			$filename = $filename . implode("", $chapNumArray) . ".xhtml";

			# allow plugins (PlantUML, Mermaid, Math...) to generate static assets
			$chapterContent = $this->prepareExportHtml($chapter['content'], $chapter['item'] ?? [], $baseurl);

			$chapterHtml = $content_start . $chapterContent . $bookEnd;

			if($prefixNumber)
			{
				$prefix = $readableChapNum . ': ';
				if($ChapterName != '')
				{
					$prefix = $ChapterName . " " . $prefix;
				}
			}

			$book->addChapter(
				$chapterName 			= $prefix . $chapter['metadata']['meta']['title'], 
				$fileName 				= $filename,
				$chapterData 			= $chapterHtml,
				$autoSplit 				= true,
				$externalReferences 	= EPub::EXTERNAL_REF_ADD
			);
		}

		$book->rootLevel();
		$book->buildTOC($cssFileName = null, $tocCSSClass = "toc", $title = $tocName);

		# Only used in case we need to debug EPub.php.
		if ($book->isLogging && isset($ebookdata['epubdebug']) && $ebookdata['epubdebug'])
		{
			$book->addChapter("Log", "Log.xhtml", $content_start . $log->getLog() . "\n</pre>" . $bookEnd);

			$book->addChapter("ePubLog", "ePubLog.xhtml", $content_start . $book->getLog() . "\n</pre>" . $bookEnd);
		}

		# Finalize the book, and build the archive.
		$book->finalize(); 

		$filename = preg_replace('/[ \.\?\!\:]/', "-", $ebookdata['title']);

		# Send the book to the client. ".epub" will be appended if missing.
		$zipData = $book->sendBook($filename);

		# After this point your script should call exit. If anything is written to the output,
		# it'll be appended to the end of the book, causing the epub file to become corrupt.
		exit();
	}
```

# 3. Helper method

Add this helper to the ebook plugin class so both `createRemotePdf` and `createEpub` can dispatch `onExportHtmlLoaded` consistently.

```php
	/**
	 * Prepare HTML for export by allowing plugins to generate static assets.
	 *
	 * Dispatches onExportHtmlLoaded with the page HTML and context.
	 *
	 * @param string $html     The HTML to prepare
	 * @param array  $item     The current page item (for page-relative context)
	 * @param string $baseurl  The base URL
	 * @return string Export-safe HTML
	 */
	private function prepareExportHtml(string $html, array $item = [], string $baseurl = ''): string
	{
		$event = new OnExportHtmlLoaded([
			'html'     => $html,
			'item'     => $item,
			'base_url' => $baseurl
		]);

		$dispatcher = $this->getDispatcher();
		$dispatcher->dispatch($event, 'onExportHtmlLoaded');

		$data = $event->getData();
		if (is_array($data))
		{
			return $data['html'] ?? $html;
		}

		return $data;
	}
```

# What changed

1. Added `use Typemill\Events\OnExportHtmlLoaded;` at the top of the plugin class.
2. Added private helper `prepareExportHtml()` that:
   - dispatches `onExportHtmlLoaded` with the current HTML and page context (`html`, `item`, `base_url`);
   - returns the modified HTML and handles both array and string payloads.
3. In `createRemotePdf()`:
   - After `$twig->fetch('@booklayouts/index.twig', $data)`, the complete HTML is passed through `prepareExportHtml()` so plugins can replace dynamic content with static assets before sending it to Kixote/WeasyPrint.
4. In `createEpub()`:
   - The imprint HTML is passed through `prepareExportHtml()` before being added to the book.
   - Each chapter's content is passed through `prepareExportHtml()` with the page item before being wrapped in the XHTML template and added to the EPUB.

The generated static assets (e.g. SVG images from PlantUML/Mermaid/Math) are stored in `/cache/generated/{plugin}/{hash}.svg` by the plugin base class helpers `generateStaticAsset()` and `getStaticAssetPath()`. For PDF, the public URLs are embedded in the HTML. For EPUB, the PHPePub library's `EXTERNAL_REF_ADD` option pulls the referenced images into the package.

# Notes

- Make sure the `ebooks` plugin class is loaded in a Typemill version that includes `system/typemill/Events/OnExportHtmlLoaded.php` and the helper methods `generateStaticAsset()` / `getStaticAssetPath()` in `system/typemill/Plugin.php`.
- Other plugins (e.g. PlantUML, Mermaid, Math) must subscribe to `onExportHtmlLoaded` and implement the export rendering path.

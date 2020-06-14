# Create And Reorder Pages

To create and reorder new pages, you can simply use the navigation on the right side of the author panel. You can choose to create a new folder or a file. You can also move and re-order items with drag & drop.

![Animation Typemill Visual Editor](media/live/tm-demo.gif){.center}

!! **Refresh the cache**
!!
!! Typemill uses its own cache-logic for the navigation in frontend and in the author interface. The navigation updates every 10 minutes when a frontend page is loaded. To manually refresh the navigation, hit the F5 button (Windows) or hard load the page (Linux or iOS). You can also delete all files inside the folder `cache` and Typemill will recreate all cached files.

## Limitations for Ordering

There are some limitations that you should know:

* The number of characters in a page name is limited to 60 right now. But you can change the navigation title in the meta-tab without limitations.
* You can move a folder within the same level, but you cannot move a folder inside another folder.

The reasons for these limitations are obvious: If you move a folder to another folder, then all urls of the folder and its files will change. This is a nightmare for your readers and for search-engines, so we decided to restrict it a bit.

## Posts for Blog- or News-Sections

As of version 1.3.3 you can change the content-type of a folder from pages to posts in the meta-tab of the folder-page. Posts are sorted by date in descending order and they cannot be reordered with drag & drop. This is why they do not appear in the navigation. Instead you will find a list of posts below the content-area of the folder. You can also create new posts there.

Good news: You can change the type of a folder whenever you want. Typemill is smart enough and rewrites all existing content of a folder and sort them by date. You can change the date manually in the meta-tab of each page. If no manual date is set, then typemill uses the create-date.


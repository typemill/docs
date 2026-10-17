# gitsync with prefixes

The GitSync plugin gets out of sync in the following cases:

* If I reorder the files in Typemill and the prefixes change or a file is moved into a folder: If I sync it with the repo, then the files are duplicated with the old files keeping the old prefix, so the moved files in Typemill get new files in the repo.
* If I rename files in the repo, then they are also created as new files in Typemill, the old ones are not removed (to be verified).


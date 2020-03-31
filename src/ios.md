# Apple iOS

## Folder Structure of an iTunes Backup

The iPhone Wiki has a great [technical documentation about the iTunes Backup folder structure](https://www.theiphonewiki.com/wiki/ITunes_Backup).
iTunes basically takes all the files on the Unix filesystem, hashes their path (prefixed by a “domain name”) using SHA1, and uses that as the file name.
There’s also the `Manifest.mbdb` file containing an index.

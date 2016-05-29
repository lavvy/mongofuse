# mongofuse
A FUSE filesystem, backed by MongoDB and written in Node.js. Mostly an exercise for me to learn MongoDB and Node.js.

### Starting
If cloned from GitHub, you'll need to run `npm install` from the directory of
your clone, then any of:
* `npm start connection-string mountpoint`
* `node index.js connection-string mountpoint`
* `./index.js connection-string mountpoint`

You can also install globally with `npm install -g mongofuse`, in which case run:
* `mongofuse connection-string mountpoint`

where `connection-string` is a MongoDB [connection string](https://docs.mongodb.com/manual/reference/connection-string/)
(maybe just `mongofuse` to use a database called `mongofuse` on an instance
running on localhost on the standard port) and `mountpoint` is an existing
directory where the filesystem is to be mounted.

### Things that work
* The root directory is now created automatically when you start mongofuse with
an empty database. This means you can actually try using it without manually
inserting stuff into MongoDB first!
* The usual reading, writing, creating, deleting, renaming etc. of files and directories
* chmod, chown, chgrp (including updating ctime)
* mknod (special files can be created, but can't be used with nodev in effect, see below)
* mtime/ctime update on file write
* symlinks
* File permissions enforced on open, truncate, ftruncate.
access function is implemented.
You can't read from a fd opened with O_WRONLY and vice versa.
You can't chmod/chown/chgrp when you shouldn't be allowed to.

### Things that don't work / aren't present (yet)
* Directory permissions are partially but not fully enforced.
There are quirks like being able to traverse or list a directory when you shouldn't,
but not permitted to find or cd into it due to the access function.
You can create and delete files in them too, as it's the directory's permissions that matter for that.
* atimes aren't updated automatically. Nor are directory mtimes/ctimes when a file is created.
* No way of specifying mount options on the command line (seems to default to nosuid, nodev)
* hardlinks
* extended attributes
* Files larger than just under 16MB, due to the maximum document size in MongoDB.
I now check for this and return EFBIG from ftruncate/truncate/write.
The solution to this is [GridFS](https://docs.mongodb.com/manual/core/gridfs/).
* I don't know if it works on OSes other than Linux, I haven't tried.
* Performance is probably pants, due to things like lack of caching,
storing the data itself within the inode document, and not explicitly
storing the filesize but counting the length of the data in getattr.

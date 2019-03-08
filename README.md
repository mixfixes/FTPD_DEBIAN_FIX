Overview≈
ftp-srv is a modern and extensible FTP server designed to be simple yet configurable.

Features
Extensible file systems per connection
Passive and active transfers
Explicit & Implicit TLS connections
Promise based API
Install
npm install ftp-srv --save

Usage
// Quick start

const FtpSrv = require('ftp-srv');
const ftpServer = new FtpSrv({ options ... });

ftpServer.on('login', (data, resolve, reject) => { ... });
...

ftpServer.listen()
.then(() => { ... });
API
new FtpSrv({options})
url
URL string indicating the protocol, hostname, and port to listen on for connections. Supported protocols:

ftp Plain FTP
ftps Implicit FTP over TLS
Note: The hostname must be the external IP address to accept external connections. 0.0.0.0 will listen on any available hosts for server and passive connections.
Default: "ftp://127.0.0.1:21"

pasv_url
The hostname to provide a client when attempting a passive connection (PASV). This defaults to the provided url hostname.

Note: If set to 0.0.0.0, this will automatically resolve to the external IP of the box.
Default: "127.0.0.1"

pasv_min
Tne starting port to accept passive connections.
Default: 1024

pasv_max
The ending port to accept passive connections.
The range is then queried for an available port to use when required.
Default: 65535

greeting
A human readable array of lines or string to send when a client connects.
Default: null

tls
Node TLS secure context object used for implicit (ftps protocol) or explicit (AUTH TLS) connections.
Default: false

anonymous
If true, will allow clients to authenticate using the username anonymous, not requiring a password from the user.
Can also set as a string which allows users to authenticate using the username provided.
The login event is then sent with the provided username and @anonymous as the password.
Default: false

blacklist
Array of commands that are not allowed.
Response code 502 is sent to clients sending one of these commands.
Example: ['RMD', 'RNFR', 'RNTO'] will not allow users to delete directories or rename any files.
Default: []

whitelist
Array of commands that are only allowed.
Response code 502 is sent to clients sending any other command.
Default: []

file_format
Sets the format to use for file stat queries such as LIST.
Default: "ls"
Allowable values:

ls bin/ls format
ep Easily Parsed LIST format
function () {} A custom function returning a format or promise for one.
Only one argument is passed in: a node file stat object with additional file name parameter
log
A bunyan logger instance. Created by default.

timeout
Sets the timeout (in ms) after that an idle connection is closed by the server
Default: 0

CLI
ftp-srv also comes with a builtin CLI.

$ ftp-srv [url] [options]
$ ftp-srv ftp://0.0.0.0:9876 --root ~/Documents
url
Set the listening URL.

Defaults to ftp://127.0.0.1:21

--root / -r
Set the default root directory for users.

Defaults to the current directory.

--credentials / -c
Set the path to a json credentials file.

Format:

[
  {
    "username": "...",
    "password": "...",
    "root": "..." // Root directory
  },
  ...
]
--username
Set the username for the only user. Do not provide an argument to allow anonymous login.

--password
Set the password for the given username.

Events
The FtpSrv class extends the node net.Server. Some custom events can be resolved or rejected, such as login.

login
ftpServer.on('login', ({connection, username, password}, resolve, reject) => { ... });
Occurs when a client is attempting to login. Here you can resolve the login request by username and password.

connection client class object
username string of username from USER command
password string of password from PASS command
resolve takes an object of arguments:

fs
Set a custom file system class for this connection to use.
See File System for implementation details.
root
If fs is not provided, this will set the root directory for the connection.
The user cannot traverse lower than this directory.
cwd
If fs is not provided, will set the starting directory for the connection
This is relative to the root directory.
blacklist
Commands that are forbidden for only this connection
whitelist
If set, this connection will only be able to use the provided commands
reject takes an error object

client-error
ftpServer.on('client-error', ({connection, context, error}) => { ... });
Occurs when an error arises in the client connection.

connection client class object
context string of where the error occurred
error error object

RETR
connection.on('RETR', (error, filePath) => { ... });
Occurs when a file is downloaded.

error if successful, will be null
filePath location to which file was downloaded

STOR
connection.on('STOR', (error, fileName) => { ... });
Occurs when a file is uploaded.

error if successful, will be null
fileName name of the file that was uploaded

RNTO
connection.on('RNTO', (error, fileName) => { ... });
Occurs when a file is renamed.

error if successful, will be null
fileName name of the file that was renamed

Supported Commands
See the command registry for a list of all implemented FTP commands.

File System
The default file system can be overwritten to use your own implementation.
This can allow for virtual file systems, and more.
Each connection can set it's own file system based on the user.

The default file system is exported and can be extended as needed:

const {FtpSrv, FileSystem} = require('ftp-srv');

class MyFileSystem extends FileSystem {
  constructor() {
    super(...arguments);
  }

  get(fileName) {
    ...
  }
}
Custom file systems can implement the following variables depending on the developers needs:

Methods
currentDirectory()
Returns a string of the current working directory
Used in: PWD

get(fileName)
Returns a file stat object of file or directory
Used in: LIST, NLST, STAT, SIZE, RNFR, MDTM

list(path)
Returns array of file and directory stat objects
Used in: LIST, NLST, STAT

chdir(path)
Returns new directory relative to current directory
Used in: CWD, CDUP

mkdir(path)
Returns a path to a newly created directory
Used in: MKD

write(fileName, {append, start})
Returns a writable stream
Options:
append if true, append to existing file
start if set, specifies the byte offset to write to
Used in: STOR, APPE

read(fileName, {start})
Returns a readable stream
Options:
start if set, specifies the byte offset to read from
Used in: RETR

delete(path)
Delete a file or directory
Used in: DELE

rename(from, to)
Renames a file or directory
Used in: RNFR, RNTO

chmod(path)
Modifies a file or directory's permissions
Used in: SITE CHMOD

getUniqueName()
Returns a unique file name to write to
Used in: STOU

Contributing
See CONTRIBUTING.md.

Contributors
OzairP
TimLuq
crabl
hirviid
DiegoRBaquero
edin-m
voxsoftware
jorinvo
Johnnyrook777
qchar
mikejestes
pkeuter
qiansc
broofa
lafin
alancnet
zgwit
License
This software is licensed under the MIT Licence. See LICENSE.

References
https://cr.yp.to/ftp.html

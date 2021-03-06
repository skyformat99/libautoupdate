# Libautoupdate

Cross-platform C library to enable your application to auto-update itself in place.

[![Build Status](https://travis-ci.org/pmq20/libautoupdate.svg?branch=master)](https://travis-ci.org/pmq20/libautoupdate)
[![Build status](https://ci.appveyor.com/api/projects/status/sjdyfwd768lh187f/branch/master?svg=true)](https://ci.appveyor.com/project/pmq20/libautoupdate/branch/master)

## API

There is only one single API, i.e. `autoupdate()`, accepting the following arguments:

- the 1st and 2nd arguments are the same as those passed to `main()`
- `host` is the host name of the update server to communicate with
- `port` is the port of the server, which is a string on Windows and a 16-bit integer on UNIX
- `path` is the paramater passed to the HTTP/1.0 HEAD request of the Round 1
- `current` is the current version string to be compared with what is returned from the server

### Round 1

Libautoupdate first makes a HTTP/1.0 HEAD request to the server, in order to peek the latest version string.

    Libautoupdate <-- HTTP/1.0 HEAD request --> Server

The server is expected to repond with `HTTP 302 Found` and provide a `Location` header.

It then compares the content of `Location` header with the current version string.
It proceeds to Round 2 if the current version string is NOT a sub-string of the `Location` header.

### Round 2

Libautoupdate makes a full HTTP/1.0 GET request to the `Location` header of the last round.

    Libautoupdate <-- HTTP/1.0 GET request --> Server

The server is expected to respond with `200 OK` transferring the new release itself.

Based on the `Content-Type` header received, an addtional inflation operation might be performed:
- `Content-Type: application/x-gzip`: Gzip Inflation is performed
- `Content-Type: application/zip`: Deflate compression is assumed and the first file is inflated and used

### Self-replacing

After all the above procedures the data is ready to proceed with a self-replacing.
The program replaces itself in-place with the help of the system temporary directory,
after which it restarts itself with the new release.

### Return Value

It never returns if a new version was detected and auto-update was successfully performed.
Otherwise, it returns one of the following integer to indicate the situation.

|  Return Value  | Indication   |
|:--------------:|--------------|
|        0       | Latest version confirmed. No need to update.
|        1       | Auto-update shall not proceed due to environment variable `LIBAUTOUPDATE_SKIP` being set. |

## Examples

Just call `autoupdate()` at the beginning of your `main()`,
before all actual logic of your application.
See the following code for examples.

### Windows

```C
#include <autoupdate.h>

int wmain(int argc, wchar_t *wargv[])
{
	int autoupdate_result;

	autoupdate_result = autoupdate(
		argc,
		wargv,
		"enclose.io",
		"80",
		"/nodec/nodec-x64.exe"
		"v1.1.0"
	);

	/* 
		actual logic of your application
		...
	*/
}
```

### UNIX

```C
#include <autoupdate.h>

int main(int argc, char *argv[])
{
	int autoupdate_result;

	autoupdate_result = autoupdate(
		argc,
		argv,
		"enclose.io",
		80,
		"/nodec/nodec-darwin-x64"
		"v1.1.0"
	);

	/* 
		actual logic of your application
		...
	*/
}
```

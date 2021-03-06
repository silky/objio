# The Object IO Library and Command

Object stores (S3, Google, Azure, Minio, Swift, etc.) all have their own
command line interfaces with their own conventions for accessing objects.
This library and command provides a simple, uniform interface to object
store facilities, both from the command line and from within Python.

# Command Line Usage

```Bash
    $ obj cat az://container/blobname
    $ obj cat gs://bucket/blobname
    $ obj cat s3://bucket/blobname
    $ obj cat file:/path
    $ cat file | obj put gs://bucket/blobname
```

# Python Usage

```Python
    import objio

    stream = objio.open("gs://bucket/blob", "rb")
```

# Configuration

The `objio` library is intended to be just a simple wrapper around
existing command line programs. You can configure how command line programs
are invoked with a YAML configuration file. Configurations are a combination
of the built-in defaults with global and directory-local overrides.

- global: ~/.objio.yaml (override with `OBJIO_GLOBAL=` environment variable)
- local: ./objio.yaml (override with `OBJIO_LOCAL=` environment variable)

The configuration file contains command specifications, either as strings
(passed to Bash for execution) or as lists (executed directly). You can override
any URL schema, and create new ones for new functionality:

```YAML
    schemes:
        http_buffered: 
            read:
                cmd: "curl --fail -L -s '{url}' --output - | mbuffer"
        random:
            read:
                cmd: ["cat", "/dev/random"]
```

After putting these definitions into `./objio.yaml`, you can say:

```Bash
    $ obj cat http_buffered://storage.googleapis.com/bucket/data
    $ obj cat random:
```

The following variables are available for substitution inside `{...}`:

        url scheme netloc path params query
        fragment username password hostname
        bucket nobucket dirname filename port

# Future Extensions

The intention is to keep this library simple and always allow command line
programs to be configured for I/O as needed by end users.

Additional functionality:

- `obj dir az://container --format csv`
- `obj del gs://container/blob`
- possibly port to Go 

Note that for Python, running I/O in a separate process is _preferable_ to using
Python-native libraries, since the latter do not run concurrently. For a Go
language implementation, protocol implementations using built in native client
libraries are potentially useful.

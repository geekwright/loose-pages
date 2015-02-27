# Configuring Cache

The cache in XOOPS can easily be configured for your specific environment.

The file cache.php is located in _var-path_/configs/ and is in [YAML](http://www.yaml.org/refcard.html) format, but wrapped in a php file for added security. The file can contain sensitive information and if _var-path_ is accessible via the seb server and/or yaml files were served as text, this information would be unprotected. The php wrapper ensures the file will be processed by PHP, and safely reveal nothing in the case of a faulty configuration.

## Default Configuration

When XOOPS is first run, it will create a configuration like the following, if no configuration file exists. It will look something like this:

```YAML
default:
    driver: FileSystem
    options:
        path: /home/user/private/xoops_data/caches/xoops_cache/stash/
        dirSplit: 2
temp:
    driver: Ephemeral
    options: {  }
```

This configuration sets up two cache definitons, default and temp. Each definition
needs a driver and options entry. The definition for default, specifies a driver type
of "FileSystem" and sets the "path" and "dirSplit" options. This definition keeps the
cached data in separate files and is a safe default for most environments. The "default"
definition will be used for all of XoopsCore's internal needs. The "path" is where in the filesystem the cache will be stored, and "dirSplit" controls how many directory levels a cache item key will be split into.

The configuration also also specifies a definition for "temp" that uses the "Ephemeral"
driver. This driver does not persist the cache across each PHP execution, but will keep
data available during a run. This can be used to save data which is costly to generate
and needs to be generated each run but may be referenced in multiple disconected places during the run. It also is handy when testing cache code, as you can save and retrieve
data, but everything is cleaned up automatically as soon as the program stops.

To reference the "temp" definiton, you specify the definition name when referencing the cache like this:
```$cache = Xoops::getInstance()->cache('temp')```

If required for your needs, you can add more definitions to the configuration file. To reference those, you specify the definition name when referencing the cache. To access a 'mycache' entry, it would look like this:
```$cache = Xoops::getInstance()->cache('mycache')```

## Other Driver Options

### SqLite

SQLite can be used as an alternative to filesystem based cache:

```YAML
default:
    driver: Sqlite
    options:
        path: /home/user/private/xoops_data/caches/xoops_cache/stash/
        extension: pdo
```

### Memcache

Memcache is an example of server based caching. Other server based caches, like Redis, have similar options. You can add additional servers by repeating the ```  - [127.0.0.1, '11211']``` line using the correct address and port.

```YAML
default:
    driver: Memcache
    options:
        servers:
            - [127.0.0.1, '11211']
        extension: memcached
```

## Stacked Drivers

Caches like memcache can be _fast_, but the lifetime for items can be short, and the available space limited. Filesystem caches can store data indefinitely and grow to huge sizes. Wouldn't it be nice if you could have the best of both? With stacked drivers, you can.

The "Composite" driver consists of multiple driver confiurations. The storage of a cache item will be managed across all the specified drivers, trasparently and automatically.

When saving an item to the cache, the entry will be written first to the first specified driver, and then propogated to each of the others in turn. When reading, the first specified driver is checked first, and if the requested entry is not found, then each remaining driver will be check in turn, until it is found, or the bottom of the drivers list is reached.

With these strategies, putting the fastest and smallest driver at the top, and the largest and slowest one at the bottom, will gain a big boost for the most commonly used cache entries without needing any code changes.

Here is a composite example, joining memcache and filesystem drivers.

```YAML
default:
    driver: Composite
    options:
        drivers:
            -
                driver: Memcache
                options:
                    servers:
                        - [127.0.0.1, '11211']
                    extension: memcached
            -
                driver: FileSystem
                options:
                    path: /home/user/private/xoops_data/caches/xoops_cache/stash/
                    dirSplit: 2

```
You can learn more about the available drivers and their options on the [Stash](http://www.stashphp.com/Drivers.html) website. We will cover a few examples here, but the underlying code using the configuration data is based on the Stash drivers.

sa-lighttpd
===========

[![Build Status](https://travis-ci.com/softasap/sa-lighttpd.svg?branch=master)](https://travis-ci.com/softasap/sa-lighttpd)


Example of usage:

Simple

```YAML

     - {
         role: "sa-lighttpd"
       }


```

Advanced

```YAML
vars:
  host_conf_properties:
    - {regexp: "^[#]?include \"/etc/lighttpd/conf-enabled/*", line: "include \"/etc/lighttpd/conf-enabled/*.conf\""}
    - {regexp: "^[#]?include \"/etc/lighttpd/sites-enabled/*", line: "include \"/etc/lighttpd/sites-enabled/*.conf\""}


    - {
         role: "sa-lighttpd",
         custom_conf_properties: "{{ host_conf_properties }}"
      }


```

Config approaches
-----------------

By default, config templated is done for compability with `sa-nginx` role approaches.
This means: default global config file, adjusted by: loads snippets from 
`conf-enabled` and `sites-enabled` ; You can have some easy to pick snippets in `conf-available` and `sites-available`
folders.


```conf

..... ORIGINAL CONFIG .....

include "/etc/lighttpd/conf-enabled/*.conf"
include "/etc/lighttpd/sites-enabled/*.conf"

```


But finally, it is up to you how you configure web server finally. You can have

ascetic one   

```conf
server.modules = (
	"mod_access",
	"mod_alias",
	"mod_compress",
 	"mod_redirect",
#       "mod_rewrite",
)

server.document-root        = "/var/www"
server.upload-dirs          = ( "/var/cache/lighttpd/uploads" )
server.errorlog             = "/var/log/lighttpd/error.log"
server.pid-file             = "/var/run/lighttpd.pid"
server.username             = "www-data"
server.groupname            = "www-data"

index-file.names            = ( "index.php", "index.html",
                                "index.htm", "default.htm",
                               " index.lighttpd.html" )

url.access-deny             = ( "~", ".inc" )

static-file.exclude-extensions = ( ".php", ".pl", ".fcgi" )

include_shell "/usr/share/lighttpd/use-ipv6.pl"

dir-listing.encoding        = "utf-8"
server.dir-listing          = "disable"
server.tag = "lighttpd"

compress.cache-dir          = "/var/cache/lighttpd/compress/"
compress.filetype           = ( "application/x-javascript", "text/css", "text/html", "text/plain" )

include_shell "/usr/share/lighttpd/create-mime.assign.pl"
include_shell "/usr/share/lighttpd/include-conf-enabled.pl"
include_shell "find /etc/lighttpd/conf.d -maxdepth 1 -name '*.conf' -exec cat {} \;"
```

combined with virtualhosts

```conf

# Server options should be set outside of host conditionals

server.port = 80
server.username = "http"
server.groupname = "http"
server.errorlog = "/var/log/lighttpd/error.log"

# Servers document-root should have a global default but also be set for each host
server.document-root = "/var/www/default/dist"


$HTTP["host"] =~ "(^|\.)example1\.com$" {
	server.document-root	= "/var/www/example1/dist"
	dir-listing.activate  = "enable"
	index-file.names      = ( "index.html" )
	mimetype.assign       = (
				".html" => "text/html",
				".txt" => "text/plain",
				".css" => "text/css",
				".js" => "application/x-javascript",
				".jpg" => "image/jpeg",
				".jpeg" => "image/jpeg",
				".gif" => "image/gif",
				".png" => "image/png",
				"" => "application/octet-stream"
				)
}
$HTTP["host"] =~ "(^|\.)example2\.com$" {
	server.document-root  = "/var/www/example2/dist"
	dir-listing.activate  = "enable"
	index-file.names      = ( "index.html" )
	mimetype.assign       = (
				".html" => "text/html",
				".txt" => "text/plain",
				".css" => "text/css",
				".js" => "application/x-javascript",
				".jpg" => "image/jpeg",
				".jpeg" => "image/jpeg",
				".gif" => "image/gif",
				".png" => "image/png",
				"" => "application/octet-stream"
				)
}

```
activate ssl

```conf

##################### enable this block to make SSL available _only_ #####################
ssl.engine = "enable" 
ssl.pemfile = "/etc/lighttpd/server.pem"
##########################################
##################### enable this block to make SSL available next to http #####################
#$SERVER["socket"] == ":443" {
#     ssl.engine                  = "enable" 
#     ssl.pemfile                 = "/etc/lighttpd/server.pem" 
# }
##########################################

```

You might want also to take a look on  https://github.com/h5bp/server-configs-lighttpd/blob/master/lighttpd.conf
```conf

# Lighttpd Server Configs | MIT License
# https://github.com/h5bp/server-configs-lighttpd

# http://redmine.lighttpd.net/projects/lighttpd/wiki

# Run as an unprivileged user
server.username = "www"
server.groupname = "www"

# Help the rc.scripts
server.pid-file = "/var/run/lighttpd/lighttpd.pid"

# A static document-root. For virtual hosting take a look at the
# mod_simple_vhost module.
server.document-root = "/var/www/sites/go/here/"

# Avoid revealing the server name and version number
server.tag = ""

# Disable directory listing
server.dir-listing = "disable"

# Modules to load
# at least mod_access and mod_accesslog should be loaded
# mod_expire should go above mod_compress (and mod_fcgi if you use it)
# otherwise expire headers will not be applied to compressed documents.
server.modules = (
    "mod_access",
    "mod_accesslog",
    "mod_redirect",
    "mod_expire",
    "mod_compress",
    "mod_setenv"
)

# Sent Response Headers
# opt-in to the future - remove meta tag from page
setenv.add-response-header = ( "X-UA-Compatible" => "IE=edge" )

# File uploads
# Make sure this folder exists and is writable to server.username
server.upload-dirs = ( "/tmp/lighttpd/uploads" )

# Where to send error-messages to
server.errorlog = "/var/log/lighttpd/error.log"

# Accesslog module
accesslog.filename = "/var/log/lighttpd/access.log"

# Compression
# Make sure this folder exists and is writable to server.username
compress.cache-dir = "/tmp/lighttpd/compress/"
compress.filetype = (
    "application/atom+xml",
    "application/javascript",
    "application/json",
    "application/ld+json",
    "application/manifest+json",
    "application/rdf+xml",
    "application/rss+xml",
    "application/schema+json",
    "application/vnd.geo+json",
    "application/vnd.ms-fontobject",
    "application/x-font-ttf",
    "application/x-javascript",
    "application/x-web-app-manifest+json",
    "application/xhtml+xml",
    "application/xml",
    "font/eot",
    "font/opentype",
    "image/bmp",
    "image/svg+xml",
    "image/vnd.microsoft.icon",
    "image/x-icon",
    "text/cache-manifest",
    "text/css",
    "text/html",
    "text/javascript",
    "text/plain",
    "text/vcard",
    "text/vnd.rim.location.xloc",
    "text/vtt",
    "text/x-component",
    "text/x-cross-domain-policy",
    "text/xml",
)

# Files to check for if .../ is requested
index-file.names = (
	"index.html",
	"index.htm",
)

# Set the event-handler (read the performance section in the manual)
# server.event-handler = "freebsd-kqueue" # needed on OS X


# Serve resources with the proper media types (f.k.a. MIME types).
# https://www.iana.org/assignments/media-types/media-types.xhtml

mimetype.assign = (

  # Data interchange

    ".atom"          =>  "application/atom+xml",
    ".geojson"       =>  "application/vnd.geo+json",
    ".json"          =>  "application/json",
    ".jsonld"        =>  "application/ld+json",
    ".map"           =>  "application/json",
    ".rdf"           =>  "application/xml",
    ".rss"           =>  "application/rss+xml",
    ".topojson"      =>  "application/json",
    ".xml"           =>  "application/xml",


  # JavaScript

    # Normalize to standard type.
    # https://tools.ietf.org/html/rfc4329#section-7.2

    ".js"            =>  "application/javascript",


  # Manifest files

    ".webapp"        =>  "application/x-web-app-manifest+json",
    ".appcache"      =>  "text/cache-manifest",
    ".webmanifest"   =>  "application/manifest+json",


  # Media files

    ".avi"           =>  "video/x-msvideo",
    ".bmp"           =>  "image/bmp",
    ".f4a"           =>  "audio/mp4",
    ".f4b"           =>  "audio/mp4",
    ".f4p"           =>  "video/mp4",
    ".f4v"           =>  "video/mp4",
    ".flv"           =>  "video/x-flv",
    ".gif"           =>  "image/gif",
    ".jpeg"          =>  "image/jpeg",
    ".jpg"           =>  "image/jpeg",
    ".m4a"           =>  "audio/mp4",
    ".m4v"           =>  "video/mp4",
    ".mov"           =>  "video/quicktime",
    ".mp3"           =>  "audio/mpeg",
    ".mp4"           =>  "video/mp4",
    ".mpeg"          =>  "video/mpeg",
    ".mpg"           =>  "video/mpeg",
    ".oga"           =>  "audio/ogg",
    ".ogg"           =>  "audio/ogg",
    ".ogv"           =>  "video/ogg",
    ".opus"          =>  "audio/ogg",
    ".png"           =>  "image/png",
    ".svg"           =>  "image/svg+xml",
    ".svgz"          =>  "image/svg+xml",
    ".wav"           =>  "audio/x-wav",
    ".wax"           =>  "audio/x-ms-wax",
    ".webm"          =>  "video/webm",
    ".webp"          =>  "image/webp",
    ".wma"           =>  "audio/x-ms-wma",
    ".wmv"           =>  "video/x-ms-wmv",
    ".xbm"           =>  "image/x-xbitmap",


    # Serving `.ico` image files with a different media type
    # prevents Internet Explorer from displaying then as images:
    # https://github.com/h5bp/html5-boilerplate/commit/37b5fec090d00f38de64b591bcddcb205aadf8ee

    ".cur"           =>  "image/x-icon",
    ".ico"           =>  "image/x-icon",


  # Web fonts

    ".woff"          =>  "application/font-woff",
    ".woff2"         =>  "application/font-woff2",
    ".eot"           =>  "application/vnd.ms-fontobject",

    # Browsers usually ignore the font media types and simply sniff
    # the bytes to figure out the font type.
    # https://mimesniff.spec.whatwg.org/#matching-a-font-type-pattern
    #
    # However, Blink and WebKit based browsers will show a warning
    # in the console if the following font types are served with any
    # other media types.

    ".otf"           =>  "font/opentype",
    ".ttc"           =>  "application/x-font-ttf",
    ".ttf"           =>  "application/x-font-ttf",


  # Other

    ".bbaw"         =>  "application/x-bb-appworld",
    ".bz2"          =>  "application/x-bzip",
    ".conf"         =>  "text/plain",
    ".crx"          =>  "application/x-chrome-extension",
    ".css"          =>  "text/css",
    ".gz"           =>  "application/x-gzip",
    ".htc"          =>  "text/x-component",
    ".htm"          =>  "text/html",
    ".html"         =>  "text/html",
    ".jar"          =>  "application/x-java-archive",
    ".log"          =>  "text/plain",
    ".oex"          =>  "application/x-opera-extension",
    ".pac"          =>  "application/x-ns-proxy-autoconfig",
    ".pdf"          =>  "application/pdf",
    ".ps"           =>  "application/postscript",
    ".qt"           =>  "video/quicktime",
    ".safariextz"   =>  "application/octet-stream",
    ".sig"          =>  "application/pgp-signature",
    ".spl"          =>  "application/futuresplash",
    ".swf"          =>  "application/x-shockwave-flash",
    ".tar"          =>  "application/x-tar",
    ".tar.bz2"      =>  "application/x-bzip-compressed-tar",
    ".tar.gz"       =>  "application/x-tgz",
    ".tbz"          =>  "application/x-bzip-compressed-tar",
    ".text"         =>  "text/plain",
    ".tgz"          =>  "application/x-tgz",
    ".torrent"      =>  "application/x-bittorrent",
    ".txt"          =>  "text/plain",
    ".vcard"        =>  "text/vcard",
    ".vcf"          =>  "text/vcard",
    ".vtt"          =>  "text/vtt",
    ".xloc"         =>  "text/vnd.rim.location.xloc",
    ".xpi"          =>  "application/x-xpinstall",
    ".xpm"          =>  "image/x-xpixmap",
    ".xwd"          =>  "image/x-xwindowdump",
    ".zip"          =>  "application/zip",


  # Default MIME type

    ""              =>      "application/octet-stream"

)

# Block access to backup and source files.
url.access-deny = ( "~", ".inc" )

$HTTP["url"] =~ "\.pdf$" {
    server.range-requests = "disable"
}

# Extensions that should not be handle via static-file transfer.
# .php, .pl, .fcgi are most often handled by mod_fastcgi or mod_cgi
static-file.exclude-extensions = ( ".php", ".pl", ".fcgi" )

# Bind to all IPs, or change to a specific IP.
server.bind = "0.0.0.0"

# Expires headers (for better cache control)
# The following expires headers are set pretty far in the future. If you don't
# control versioning with filename-based cache busting, consider lowering the
# cache time for resources like CSS and JS to something like 1 week.

# CSS
$HTTP["url"] =~ ".css" {
    expire.url = ( "" => "access plus 1 years" )
}

# Data interchange
$HTTP["url"] =~ ".(json|xml)" {
    expire.url = ( "" => "access plus 0 seconds" )
}

# Favicon
$HTTP["url"] =~ ".ico" {
    expire.url = ( "" => "access plus 7 days" )
}

# HTML components (HTCs)
$HTTP["url"] =~ ".htc" {
    expire.url = ( "" => "access plus 1 months" )
}

# HTML
$HTTP["url"] =~ ".html" {
    expire.url = ( "" => "access plus 0 seconds" )
}

# JavaScript
$HTTP["url"] =~ ".js" {
    expire.url = ( "" => "access plus 1 years" )
}

# Manifest files
$HTTP["url"] =~ ".(appcache|manifest|webapp)" {
    expire.url = ( "" => "access plus 0 seconds" )
}

$HTTP["url"] =~ ".webmanifest" {
    expire.url = ( "" => "access plus 1 week" )
}

# Media
$HTTP["url"] =~ ".(gif|jpg|jpeg|png|m4a|f4a|f4b|oga|ogg|webm)" {
    expire.url = ( "" => "access plus 1 months" )
}

# Web feeds
$HTTP["url"] =~ ".(atom|rss)" {
    expire.url = ( "" => "access plus 1 hours" )
}

# Web fonts
$HTTP["url"] =~ ".(eot|otf|svg|svgz|ttf|ttc|woff)" {
    expire.url = ( "" => "access plus 1 months" )
}

# Default
expire.url = ( "" => "access plus 1 months" )

```


Usage with ansible galaxy workflow
----------------------------------

If you installed the `sa-lighttpd` role using the command


`
   ansible-galaxy install softasap.sa-lighttpd
`

the role will be available in the folder `library/softasap.sa-lighttpd`
Please adjust the path accordingly.

```YAML

     - {
         role: "softasap.sa-lighttpd"
       }

```


Misc
----

Note, that Debian install way differs from centos one

Scope that additionally processed on centos systems

Folders under control

```
/var/log/lighttpd 
/var/run/lighttpd 
/var/cache/lighttpd


/var/www
/var/log/lighttpd
/var/cache/lighttpd/compress
/var/cache/lighttpd/uploads
/etc/lighttpd/conf-available
/etc/lighttpd/conf-enabled


```

Notes on default init

```
        if [ ! -r /var/www/index.lighttpd.html ];
        then
                cp /usr/share/lighttpd/index.html /var/www/index.lighttpd.html
        fi
        mkdir -p /var/run/lighttpd > /dev/null 2> /dev/null
        chown www-data:www-data /var/log/lighttpd /var/run/lighttpd
        chown www-data:www-data /var/cache/lighttpd /var/cache/lighttpd/compress /var/cache/lighttpd/uploads
        chmod 0750 /var/log/lighttpd /var/run/lighttpd
```

```
debian/lighttpd.conf                        /etc/lighttpd
debian/conf-available/05-auth.conf          /etc/lighttpd/conf-available
debian/conf-available/10-status.conf        /etc/lighttpd/conf-available
debian/conf-available/10-cgi.conf           /etc/lighttpd/conf-available
debian/conf-available/10-fastcgi.conf       /etc/lighttpd/conf-available
debian/conf-available/10-proxy.conf         /etc/lighttpd/conf-available
debian/conf-available/10-rrdtool.conf       /etc/lighttpd/conf-available
debian/conf-available/10-simple-vhost.conf  /etc/lighttpd/conf-available
debian/conf-available/10-ssi.conf           /etc/lighttpd/conf-available
debian/conf-available/10-ssl.conf           /etc/lighttpd/conf-available
debian/conf-available/10-userdir.conf       /etc/lighttpd/conf-available
debian/conf-available/README                /etc/lighttpd/conf-available
debian/create-mime.assign.pl                /usr/share/lighttpd/
debian/include-conf-enabled.pl              /usr/share/lighttpd/
````


Copyright and license
---------------------

Code is dual licensed under the [BSD 3 clause] (https://opensource.org/licenses/BSD-3-Clause) and the [MIT License] (http://opensource.org/licenses/MIT). Choose the one that suits you best.

Reach us:

Subscribe for roles updates at [FB] (https://www.facebook.com/SoftAsap/)

Join gitter discussion channel at [Gitter](https://gitter.im/softasap)

Discover other roles at  http://www.softasap.com/roles/registry_generated.html

visit our blog at http://www.softasap.com/blog/archive.html 

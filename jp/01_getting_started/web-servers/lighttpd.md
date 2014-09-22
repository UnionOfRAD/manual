# li3 On Lighty

This quick guide is meant to help Lighty users get a development setup running in order to create li3 apps served up by Lighttpd. Lighttpd is a web server designed and optimized for high-performance production environments. It has a small memory footprint, yet contains a rich feature set. If you're planning on using li3 and Lighttpd together, this guide is for you.

## Running Lighty

You can download a copy of Lighty on the [official website](http://www.lighttpd.net/download/). Once downloaded, follow the instructions to get a running instance.

Installation details should be found in the `INSTALL` file packaged with the downloaded archive. Online installation tips can also be found on [the Lighttpd website](http://redmine.lighttpd.net/projects/lighttpd/wiki/InstallFromSource).

## Configuration

Let us assume for this example that you're developing locally and would like your app to be accessible at `http://lithium.local/li3`.

First, enable `mod_magnet` in your `lighttpd.conf`.

```
	server.modules += ( "mod_magnet" )
```

Then, save the following script in a file named `li3.lua`, preferably somewhere near your `lighttpd.conf`.

```
	-- Helper function
	function file_exists(path)
	  local attr = lighty.stat(path)
	  if (attr) then
	      return true
	  else
	      return false
	  end
	end
	function removePrefix(str, prefix)
	  return str:sub(1,#prefix+1) == prefix.."/" and str:sub(#prefix+2)
	end

	--[[
	  Prefix without the trailing slash.
	  If you are _not_ serving out of a subfolder, leave `prefix` empty.
	--]]
	local prefix = '/li3'

	-- The magic ;)
	if (not file_exists(lighty.env["physical.path"])) then
	    -- File still missing: pass it to the fastCGI backend.
	    request_uri = removePrefix(lighty.env["uri.path"], prefix)
	    if request_uri then
	      lighty.env["uri.path"]          = prefix .. "/index.php"
	      local uriquery = lighty.env["uri.query"] or ""
	      lighty.env["uri.query"] = uriquery .. (uriquery ~= "" and "&" or "") .. "url=" .. request_uri
	      lighty.env["physical.rel-path"] = lighty.env["uri.path"]
	      lighty.env["request.orig-uri"]  = lighty.env["request.uri"]
	      lighty.env["physical.path"]     = lighty.env["physical.doc-root"] .. lighty.env["physical.rel-path"]
	    end
	end
	-- Fallthrough will put it back into the lighty request loop..
	-- That means we get the 304 handling for free. ;)
```

Finally, in your `lighttpd.conf`, add the following conditional:

```
	$HTTP["host"] =~ "lithium.local" {
		server.document-root = "/path/to/your/app/webroot/"
		magnet.attract-physical-path-to = ( "/path/to/li3.lua" )
	}
```

You'll probably need to add a line item in your `/etc/hosts` file as well:

```
	127.0.0.1 lithium.local
```

Restart your lighttpd process, and you're done!
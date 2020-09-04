# Content-Encoding Instructions
This repo has some discussions about how to set Content-Encoding for Apache, S3, IIS


## The Content-Encoding Warning: 

When running WebViewer on my server, you may encounter the warning "Your server has not been configured to serve .gz and .br files with the expected Content-Encoding. See [http://www.pdftron.com/kb_content_encoding](http://www.pdftron.com/kb_content_encoding) for instructions on how to resolve this." 


## What does this mean and how do I fix this?
WebViewer contains certain large files that are already compressed using brotli (abbreviated br) or gzip encoding. As the warning suggests for ideal performance your server should be adjusted to serve these files with the HTTP Content-Encoding header. (see [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding) for more details on Content-Encoding).

Without the Content-Encoding header, some issues are:
  - Lack of WASM streaming compilation - being able to download and compile the code in parallel (while it is downloading)
  - Lack of WASM caching. (For subsequent loads of the page) 

Note that the WASM mime type('Content-Type': 'application/wasm') also needs to be served correctly for these. 
Also, with this header the browser can quickly decompress these files using native code. Note that WebViewer can still function by decompressing these files in JavaScript, but this may have an impact on loading speed.

### For fixing:
1. Note that in order to serve files with `Content-Encoding: br` your site must use `HTTPS` rather than `HTTP`. This is due to behavior in certain browsers (in particular Chrome) that leads them to reject `brotli encoding` served over HTTP.
2. The goal is to serve files within WebViewer containing `.gz` in their file name with `Content-Encoding: gzip` and files containing `.br` in their file name with `Content-Encoding: br`.

### Specific Instructions for different Servers:
- For **Apache servers**: 
  1. WebViewer includes an `.htaccess` file (under `WebViewer/lib/core`) to implement this logic, so in most cases the only necessary step is to enable the mod_headers module. Note that this will only work if `.htaccess` is enabled for your particular server.
  2. Note: you'll only want to add the Content-Encoding header if the filename includes `.gz` or `.br` as shown in the example htaccess file.
  3. A sample `.htaccess` file has been provided at "Apache/.htaccess"

- For **S3**: 
  1. Using a tool such as Cloudberry (S3 browser should also have similar functionality) you can directly set the Content-Encoding of a particular file.
  2. Clicking other mouse button on the file to pop up the context menu -> Set HTTP headers -> Enter Content-Encoding and either gzip or br depending on the file. 
  3. S3 also provides scripting APIs that can accomplish the same result when uploading if this needs to be done frequently.

- For **IIS**:
  1. If the url rewrite module is not available/installed you can install it from [here](https://www.iis.net/downloads/microsoft/url-rewrite). (Here are is a [example](http://web-site-scripts.com/knowledge-base/article/AA-00470/0) that discusses installing it). 
  2. Put the file `web.config` in the WebViewer/lib/core folder. (A sample has been provied at "IIS/web.config")
  3. Note that you may also need to restart your server, here is a [stackoverflow page](https://stackoverflow.com/questions/18261507/change-to-web-config-on-server-is-not-going-into-effect.) discuss about changing the web.config file on server.
  4. This has been tested on IIS 10, though this should work on IIS 7 and above. (web.config is only supported on IIS 7+)

- Coming on the way:
  - guide for Nginx


### Some useful links:
 - pdfnet-webviewer google group discussion: https://groups.google.com/g/pdfnet-webviewer/c/SeVvEpPAELg/m/bI4FcBy3BQAJ
 - Web FAQ troubleshooting: https://www.pdftron.com/documentation/web/faq/content-encoding/





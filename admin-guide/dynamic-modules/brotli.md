# Brotli

Brotli is a general‑purpose, lossless data compression algorithm that uses a variant of the LZ77 algorithm, Huffman coding, and second‑order context modeling. Its compression ratio is comparable to the best currently available general‑purpose compression methods. Its speed is similar to [DEFLATE](https://www.ietf.org/rfc/rfc1951.txt) but with denser compression.

### Installation Instructions

1. Install the Brotli module.

   For Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-brotli
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-brotli
   ```

2. Put the following directive in the top‑level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_brotli_filter_module.so;
   load_module modules/ngx_http_brotli_static_module.so;
   ```

3. Perform additional configuration as required by the [module](https://github.com/eustas/ngx_brotli/).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```


Provide your solution here:
- First of all, we need to check number of processes and memory usage by nginx
```sh
ps -aux | grep nginx
```
or with `top` or `htop` command.
- Check the nginx log (both access and error log) to address the problem.
- Out of memory problems may come from difference reasons, there are some typical cases to consider:
  - **High traffic volume**: Your webserver must handle too much concurrent connections at the same time (may be it is the DoS or brute-force attack)
  - **Cache too large**: The caching configuration was configured improperly or you forget to set time-to-live for cache content.
  - **Uncompressed content**: Nginx send original size content to client, leading to large memory usage
- Solution:
  - Consider use Nginx rate limit to prevent and mitigate DoS attack (test carefully before applying to production because it may impact to normal users):
    ```ini
      http {
          limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

          server {
              location /search/ {
                  limit_req zone=one burst=5 nodelay;
              }
          }
      }
    ```
    - `limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;` creates a zone named "one" with a size of 10MB and a limit of 1 request per second per IP address.
    - `limit_req zone=one burst=5 nodelay;` allows bursts of up to 5 requests, but no delay is added for these bursts.

  - Integrate with IDS/IPS tool to block macilious IP(i.e Fail2ban, Crowder)
  - Use caching configuration efficiently, limitting the size and life cycle for cache content. For example:
    ```ini
      http {
          proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m use_temp_path=off;

          server {
              location / {
                  proxy_cache my_cache;
                  proxy_pass http://your_backend;
              }
          }
      }
    ```
  - Utilize compression to reduce transferred content to client, reduce resource usage by Nginx.
    ```ini
      http {
        server {
          ...
        }
        gzip on;
        gzip_types text/css application/javascript application/json image/svg+xml;
        gzip_comp_level 9;
      }
    ```
  - Increase server size: Well, if the
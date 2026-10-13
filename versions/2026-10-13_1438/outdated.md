# Outdated patterns (not to be included as rules)

## Codebase

### General

#### validation/security

- Use Gate::after to override earlier authorization decisions or enforce non-bypassable administrator constraints — Laravel 13 after-hook return values cannot override a non-null allow or deny, so mandatory constraints belong in the decision path — sources: 219-B2M3FCjRz6w https://www.youtube.com/watch?v=B2M3FCjRz6w&t=930s, https://laravel.com/docs/13.x/authorization

## Infra

### scaling/Octane/performance

- Use HTTP/2 server push to send anticipated page resources before the browser requests them, with preload Link headers informing server push — Chrome disabled HTTP/2 server push by default in version 106 and Nginx made its push directives obsolete; preload/Early Hints are distinct mechanisms — sources: 064-eq14CaVAjk8 <https://speakerdeck.com/ramsey/http-is-dead-long-live-http2-laracon-2016> (timestamp unavailable), <https://laravel.com/docs/13.x/vite>, <https://developer.chrome.com/blog/removing-push>, <https://nginx.org/en/docs/http/ngx_http_v2_module.html>

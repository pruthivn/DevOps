# AWS Scenarios

Document AWS DevOps scenarios, troubleshooting cases, and practice examples here.

## 1. what is cache busting?
A. Cache busting is a web development technique that forces browsers to fetch the latest version of a file rather than using an outdated, locally cached copy.

If you update your website's code, returning visitors might still see the old version because their browser believes it already has the correct file. By changing the file's URL—such as modifying style.css to style.css?v=2 or style.a1b2c3.css—the browser treats it as a completely new resource and bypasses the cache to download the updated file from the server.

**Common Cache Busting Methods:**

**File Hashing (Recommended):** Build tools (like Webpack, Vite, or Gulp) automatically rename files based on their content (e.g., app.892jf9.js). Whenever you change the code, the hash changes, ensuring the browser fetches the new file.

**Query Strings:** Adding parameters like ?v=1.2 or a timestamp to the end of your file's URL in your HTML. While easy to implement, some proxies and CDNs struggle to cache files with query strings effectively.

**File Path Versioning:** Changing the directory structure (e.g., /v1/style.css to /v2/style.css) to signal an update.
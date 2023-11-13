# Challenge link
URL:      [http://45.122.249.68:20021/](http://45.122.249.68:20021/) (maybe the server was shut down)

# Challenge description
Let's get started. Try to read the flag in the root directory.

# Challenge source code
Source code on the page:
  ```
  <?php

  error_reporting(0);

  show_source(__FILE__);

  function check_valid($str)
  {
    $blacklist = ['php', 'file', 'glob', 'data', 'http', 'zip', 'zlib', 'phar', 'W1'];
    $pattern = '/' . implode('|', $blacklist) . '/i';

    if (preg_match($pattern, $str, $matches)) {
        return false;
    }
    return true;
  }

  if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $body = file_get_contents('php://input');
    $json = json_decode($body, true);

    if ($json === null && json_last_error() !== JSON_ERROR_NONE) {
        header('Content-Type: application/json');
        echo json_encode(['error' => 'Invalid JSON']);
        exit;
    }


    if (isset($json) && isset($json['page']) && check_valid($body)) {
        $page = $json['page'];
        $content = file_get_contents($page);
        if (!$content) {
            $content = "Not found";
        } else {
            if (!check_valid($content)) {
                $content = "Invalid content";
            }
        }
    } else {
        $content = "Invalid request";
    }

    header('Content-Type: application/json');
    echo json_encode(['content' => $content]);
  }
  ```
# Difficulty
Easy

# Author
dcduc

# Approach
- The first thing which I saw when I connect to the site is the source code:
  ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/f563d0dc-900c-4398-aac4-47d5a7fcaf82)

- This is a PHP web application. Some words were blocked:
  ```
  function check_valid($str)
  {
    $blacklist = ['php', 'file', 'glob', 'data', 'http', 'zip', 'zlib', 'phar', 'W1'];
    $pattern = '/' . implode('|', $blacklist) . '/i';

    if (preg_match($pattern, $str, $matches)) {
        return false;
    }
    return true;
  }
  ```
- It also has POST method:
  ```
  if ($_SERVER['REQUEST_METHOD'] === 'POST') {
  ...
  ```
- And a JSON object called "page":
  ```
  if (isset($json) && isset($json['page']) && check_valid($body)) {
  ...
  ```
- I put the site into Burp Suite and tried to change it to the POST method:
  ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/5376a4c5-3c61-4fea-a757-dd7487c236bb)

  Then add a required JSON object:
  ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/e4ee58cc-01d9-4cf2-84cd-ccf7e8031e9b)

- At first, I thought it may be a kind of vulnerability relevant to JSON data or file transmission because of some blocked keywords.
- I see a code line that I do not understand:
  ```
  $body = file_get_contents('php://input');
  ...
  ```
- file_get_contents() is a function in PHP that gets the file's contents and returns it to the caller. But what does "php://input" mean?
- Since I don't know anything about it, I searched it:
  ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/d8632081-c5c9-4619-a1e7-60da49dc0179)

- But I saw an interesting thing: the PHP manual page has manuals for some keywords that were blocked (I circled it on the image). Maybe something interesting with these keywords.
- I tried to search for vulnerabilities related to them and saw this article: [link](https://medium.com/@Aptive/local-file-inclusion-lfi-web-application-penetration-testing-cc9dc8dd3601)
- Now I know that this challenge has local file inclusion vulnerability.

# Problem-solving
  ## The payload
  - Back to the JSON object mentioned above. Since I don't know what it is used for, I read again the source code:
    ```
     if (isset($json) && isset($json['page']) && check_valid($body)) {
        $page = $json['page'];
        $content = file_get_contents($page);
        if (!$content) {
            $content = "Not found";
        } else {
            if (!check_valid($content)) {
                $content = "Invalid content";
            }
        }
    } else {
        $content = "Invalid request";
    }

    header('Content-Type: application/json');
    echo json_encode(['content' => $content]);
    ```
  - The code means that the server takes JSON object "page", performs the PHP command *file_get_contents($page)*, and sends back the content. I thought that I could assign a command to the *page* object and send it to the server.
  - Reading the article carefully gives me a simple payload:
    ```
    php://filter/convert.base64-encode/resource=/etc/passwd
    ```
  - This is a PHP command to read and return the content of a directory or file. Now I open Burp Suite and assign it to "page":
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/567c8360-c3a5-40e6-bdd6-d1e72bb1aa5f)

  - I forgot that *php* was blocked. So I choose to encode a *p* character using Unicode:
    ```
    \u0070hp://filter/convert.base64-encode/resource=/etc/passwd
    ```
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/a9ccba0a-16fe-4828-9b04-e6b94a9ae936)

  - It worked!!
  - **But why it worked? Why do I have to encode as Unicode, not Hexadecimal or another type?** The reason is based on the source code:
    ```
    ...
    if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $body = file_get_contents('php://input');
    $json = json_decode($body, true);
    ...
    ```
  - Function json_decode() in PHP will decode a JSON string and return a JSON object. During the decode session, it also decodes all Unicode characters if exist. For example:
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/3caf7b24-54ab-417f-8676-970d8f7e7588)
    
  - The filter will be fooled that I didn't use *php*. Now try to read the flag file:
    ![image](https://github.com/NoSpaceAvailable/WannagameFreshman2023/assets/143888307/cea38df7-ed07-43fb-bb83-8232c3414968)

  - Decode it as base64 to get the flag.
  - Flag: W1{w3lc0m3_w3b_w4rrj0rs}

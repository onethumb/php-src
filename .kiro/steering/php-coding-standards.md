---
inclusion: always
---

# PHP Core Development Coding Standards

This steering document provides comprehensive coding standards and best practices for PHP core development, derived from analysis of the PHP source code, the official CODING_STANDARDS.md, and the ext/hash extension implementation patterns.

## Code Implementation Standards

### Language and Compatibility
- PHP is implemented in **C11** standard
- Use fixed-width integers from stdint.h (int8_t, int16_t, int32_t, int64_t and unsigned counterparts)
- Maintain compatibility across platforms (Windows, Linux, BSD, Darwin, Solaris)

### Memory Management
- **ALWAYS** use PHP memory functions: `emalloc()`, `efree()`, `estrdup()`, `ecalloc()`, `erealloc()`
- **NEVER** use standard C library functions (`malloc()`, `free()`, `strdup()`) unless interfacing with third-party libraries
- Memory returned to the engine **MUST** be allocated using `emalloc()`
- Use `malloc()` only when third-party libraries need to control memory or memory must survive between requests

### String Handling
- Remember that PHP strings have length properties - don't calculate with `strlen()`
- Write binary-safe functions that use the length property
- Functions that modify strings should return the new length
- **NEVER USE `strncat()`** - if absolutely necessary, check man page twice and consider alternatives

### Resource Management
- Functions given pointers to resources should **NOT** free them
- Exceptions: function's designated behavior is freeing, boolean argument controls freeing, or low-level parser routines
- Document tightly integrated functions and declare them `static` when possible

## Naming Conventions

### User-Level Functions
```c
// Good examples
PHP_FUNCTION(str_word_count)
PHP_FUNCTION(array_key_exists)
PHP_FUNCTION(hash_hmac_file)

// Bad examples  
PHP_FUNCTION(hw_GetObjectByQueryCollObj)  // Mixed case, unclear
PHP_FUNCTION(pg_setclientencoding)        // No underscores
PHP_FUNCTION(jf_n_s_i)                   // Meaningless abbreviations
```

**Rules:**
- Use `PHP_FUNCTION()` macro
- Lowercase with underscores
- Minimize letter count while maintaining readability
- Avoid abbreviations that decrease readability
- Use `parent_*` pattern for function families (e.g., `hash_*`, `array_*`)

### Internal Functions
```c
// External API functions
PHPAPI char *php_session_create_id(PS_CREATE_SID_ARGS);
PHPAPI void php_hash_register_algo(const char *algo, const php_hash_ops *ops);

// Internal module functions (static)
static int php_session_destroy()
static void php_hash_init_context(php_hashcontext_object *hash)
```

**Rules:**
- External API: `php_modulename_function()` format
- Declare in `php_modulename.h`
- Internal functions: `static` and not in header
- Main source file: `modulename.c`
- Main header file: `php_modulename.h`

### Variables and Parameters
```c
// Good
int hash_algorithm_count;
const char *algorithm_name;
php_hashcontext_object *hash_context;

// Bad
int hac;           // Meaningless abbreviation
char *n;           // Single letter (except loop counters)
int HashCount;     // Wrong case
```

**Rules:**
- Lowercase with underscores
- Meaningful names (except trivial loop counters like `i`)
- Avoid single-letter variables except for loops

### Classes and Methods
```php
// Good class names (PascalCase)
class HashContext {}
class CurlResponse {}
class HttpStatusCode {}
class Ssl\Certificate {}

// Good method names (camelCase)
public function getData() {}
public function performHttpRequest() {}
public function buildSomeWidget() {}

// Bad examples
class curl_response {}     // Wrong case
class HTTPStatusCode {}    // All caps acronym
public function get_Data() {} // Mixed styles
```

## Code Structure and Style

### Indentation and Formatting
- Use **K&R style** braces
- Use **tab characters** for indentation (4 spaces equivalent)
- Maintain generous whitespace between logical sections
- One empty line between variable declarations and statements
- At least one empty line between functions (preferably two)

```c
// Good formatting
if (foo) {
    bar;
}

// Bad formatting  
if(foo)bar;
```

### Preprocessor Directives
```c
// Correct - # at column one
#ifdef HAVE_CONFIG_H
#include <config.h>
#endif

#if defined(PHP_WIN32)
#    define PHPAPI __declspec(dllexport)
#else
#    define PHPAPI
#endif
```

### Macros and Constants
- Use `#define` for all numeric constants and behavioral flags
- Use meaningful names for all constants
- Prefer `PHP_*` macros in PHP source, `ZEND_*` in Zend source
- Use `strlen()` for string literal lengths instead of `sizeof()-1`

```c
// Good
#define PHP_HASH_HMAC    0x0001
#define PHP_HASH_VERSION PHP_VERSION

// Usage
if (flags & PHP_HASH_HMAC) {
    // HMAC processing
}
```

## Function Design Patterns

### Return Types
```c
// Use bool for yes/no questions
bool php_hash_algo_exists(const char *algo);

// Use zend_result for operations that may succeed/fail  
zend_result php_hash_init_context(php_hashcontext_object *hash);

// Return new length for string modification functions
size_t php_hash_update_context(php_hashcontext_object *hash, const char *data, size_t len);
```

### Function Signatures
```c
// Good - clear parameter types and purposes
PHPAPI php_hashcontext_object *php_hash_init(
    const char *algo, 
    zend_long flags, 
    const char *key, 
    size_t key_len
);

// Function that doesn't exist should not be defined
// Use function_exists() for runtime checks instead
```

## Extension Structure Patterns

### File Organization (based on ext/hash)
```
ext/modulename/
├── modulename.c              # Main implementation
├── php_modulename.h          # Public API header  
├── modulename.stub.php       # Function signatures for arginfo
├── modulename_arginfo.h      # Generated argument info
├── config.m4                 # Unix build configuration
├── config.w32                # Windows build configuration
├── CREDITS                   # Author information
├── tests/                    # Test files
│   ├── basic_001.phpt
│   ├── error_001.phpt
│   └── ...
└── submodules/               # Algorithm-specific implementations
    ├── modulename_algo1.c
    ├── php_modulename_algo1.h
    └── ...
```

### Extension Registration Pattern
```c
// In modulename.c
zend_module_entry modulename_module_entry = {
    STANDARD_MODULE_HEADER,
    "modulename",
    ext_functions,
    PHP_MINIT(modulename),
    PHP_MSHUTDOWN(modulename),
    NULL,
    NULL,
    PHP_MINFO(modulename),
    PHP_MODULENAME_VERSION,
    STANDARD_MODULE_PROPERTIES
};

// Initialization function
PHP_MINIT_FUNCTION(modulename)
{
    // Register classes, constants, etc.
    return SUCCESS;
}
```

## Testing Standards

### Test File Naming
- Use `.phpt` extension for all tests
- Descriptive names: `hash_hmac_basic.phpt`, `hash_error_invalid_algo.phpt`
- Group by functionality: `basic`, `error`, `edge_cases`

### Test Structure
```php
--TEST--
Hash: hash_hmac() basic functionality
--FILE--
<?php
echo "*** Testing hash_hmac() basic functionality ***\n";

$data = "The quick brown fox jumps over the lazy dog";
$key = "secret";

var_dump(hash_hmac('md5', $data, $key));
var_dump(hash_hmac('sha1', $data, $key));
?>
--EXPECT--
*** Testing hash_hmac() basic functionality ***
string(32) "80070713463e7749b90c2dc24911e275"
string(40) "de7c9b85b8b78aa6bc8a7a36f70a90701c9db4d9"
```

### Error Testing Pattern
```php
--TEST--
Hash: hash() error conditions
--FILE--
<?php
try {
    hash('invalid_algo', 'data');
} catch (ValueError $e) {
    echo $e->getMessage() . "\n";
}
?>
--EXPECT--
hash(): Argument #1 ($algo) must be a valid hashing algorithm
```

## Documentation Standards

### Code Comments
```c
/* {{{ php_hash_init
 * Initialize a hash context with the specified algorithm
 * Returns SUCCESS on success, FAILURE on error
 */
PHPAPI zend_result php_hash_init(php_hashcontext_object *hash, const char *algo)
{
    const php_hash_ops *ops;
    
    if (!(ops = php_hash_fetch_ops(algo))) {
        return FAILURE;
    }
    
    hash->ops = ops;
    hash->context = emalloc(ops->context_size);
    ops->hash_init(hash->context, NULL);
    
    return SUCCESS;
}
/* }}} */
```

### Header Documentation
```c
/**
 * Hash algorithm operations structure
 * Defines the interface for hash algorithm implementations
 */
typedef struct _php_hash_ops {
    const char *algo;                           /* Algorithm name */
    php_hash_init_func_t hash_init;           /* Initialization function */
    php_hash_update_func_t hash_update;       /* Update function */
    php_hash_final_func_t hash_final;         /* Finalization function */
    size_t digest_size;                        /* Output size in bytes */
    size_t block_size;                         /* Block size for algorithm */
    size_t context_size;                       /* Context structure size */
    unsigned is_crypto: 1;                     /* Cryptographic quality flag */
} php_hash_ops;
```

## Security Considerations

### Sensitive Parameters
```php
// Use SensitiveParameter attribute for security
function hash_hmac(
    string $algo, 
    string $data, 
    #[\SensitiveParameter] string $key,  // Marked as sensitive
    bool $binary = false
): string {}
```

### Input Validation
```c
// Always validate input parameters
if (!algo || !*algo) {
    zend_argument_value_error(1, "must be a valid hashing algorithm");
    RETURN_THROWS();
}

// Check algorithm existence
if (!php_hash_fetch_ops(algo)) {
    zend_argument_value_error(1, "must be a valid hashing algorithm");  
    RETURN_THROWS();
}
```

## Platform-Specific Code

### Windows Compatibility
```c
#ifdef PHP_WIN32
    // Windows-specific implementation
    #define PHPAPI __declspec(dllexport)
    #include "win32/php_win32_globals.h"
#else
    // Unix-like systems
    #define PHPAPI __attribute__ ((visibility("default")))
#endif
```

### Endianness Handling
```c
// Use proper byte order conversion
static void encode_uint32(unsigned char *output, uint32_t input)
{
    output[0] = (unsigned char)(input & 0xff);
    output[1] = (unsigned char)((input >> 8) & 0xff);  
    output[2] = (unsigned char)((input >> 16) & 0xff);
    output[3] = (unsigned char)((input >> 24) & 0xff);
}
```

## Performance Considerations

### Efficient String Operations
```c
// Use smart strings for dynamic content
smart_str result = {0};
smart_str_appendl(&result, data, data_len);
smart_str_0(&result);  // Null terminate
RETURN_STR(result.s);
```

### Memory Efficiency
```c
// Pre-allocate known sizes
char *buffer = emalloc(expected_size);

// Use appropriate data structures
HashTable *algo_table = &php_hash_hashtable;
```

This steering document should be referenced for all PHP core development work to ensure consistency with established patterns and maintain code quality standards.
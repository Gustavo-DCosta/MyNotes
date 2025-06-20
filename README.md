# C Programming Cheatsheet
## Memory Management & Advanced Topics

## Table of Contents
1. [Memory Management](#memory-management)
2. [Pointers](#pointers)
3. [Character Management](#character-management)
4. [String Management](#string-management)
5. [Arrays](#arrays)
6. [Hash Tables](#hash-tables)
7. [File Operations](#file-operations)
8. [Advanced Memory Concepts](#advanced-memory-concepts)
9. [Theoretical Foundations](#theoretical-foundations)

---

## Memory Management

### Stack vs Heap
```c
// Stack allocation (automatic)
int stack_var = 42;
char buffer[1024];

// Heap allocation (dynamic)
int *heap_var = malloc(sizeof(int));
char *dynamic_buffer = malloc(1024);

// Always free heap memory
free(heap_var);
free(dynamic_buffer);
```

### Memory Allocation Functions
```c
#include <stdlib.h>

// Basic allocation
void *malloc(size_t size);
void *calloc(size_t num, size_t size);  // Zero-initialized
void *realloc(void *ptr, size_t new_size);
void free(void *ptr);

// Example usage
int *arr = malloc(10 * sizeof(int));
int *zero_arr = calloc(10, sizeof(int));
arr = realloc(arr, 20 * sizeof(int));
free(arr);
```

### Memory Safety Best Practices
```c
// Always check allocation success
int *ptr = malloc(sizeof(int) * 100);
if (ptr == NULL) {
    fprintf(stderr, "Memory allocation failed\n");
    exit(1);
}

// Avoid double free
free(ptr);
ptr = NULL;  // Prevent accidental reuse

// Avoid memory leaks
void cleanup_function() {
    if (ptr != NULL) {
        free(ptr);
        ptr = NULL;
    }
}
```

---

## Pointers

### Pointer Basics
```c
int value = 42;
int *ptr = &value;        // Pointer to value
int **double_ptr = &ptr;  // Pointer to pointer

printf("Value: %d\n", value);
printf("Via pointer: %d\n", *ptr);
printf("Via double pointer: %d\n", **double_ptr);
```

### Pointer Arithmetic
```c
int arr[] = {1, 2, 3, 4, 5};
int *p = arr;

// Moving through array
printf("%d\n", *p);      // 1
printf("%d\n", *(p+1));  // 2
printf("%d\n", *++p);    // 2 (increment then dereference)
printf("%d\n", *p++);    // 2 (dereference then increment)
```

### Function Pointers
```c
// Function pointer declaration
int (*operation)(int, int);

// Functions
int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

// Usage
operation = add;
int result = operation(5, 3);  // 8

operation = multiply;
result = operation(5, 3);      // 15
```

### Void Pointers and Generic Programming
```c
void swap(void *a, void *b, size_t size) {
    char temp[size];
    memcpy(temp, a, size);
    memcpy(a, b, size);
    memcpy(b, temp, size);
}

// Usage
int x = 5, y = 10;
swap(&x, &y, sizeof(int));
```

---

## Character Management

### Character Operations
```c
#include <ctype.h>

char c = 'A';
printf("isalpha: %d\n", isalpha(c));
printf("isdigit: %d\n", isdigit(c));
printf("toupper: %c\n", toupper('a'));
printf("tolower: %c\n", tolower('A'));
```

### Character Arrays vs Strings
```c
// Character array (not null-terminated)
char chars[] = {'H', 'i'};

// String (null-terminated)
char string[] = "Hi";
char *str_ptr = "Hello";

// Manual null termination
char manual[4];
manual[0] = 'H';
manual[1] = 'i';
manual[2] = '\0';
```

---

## String Management

### String Functions
```c
#include <string.h>

char src[] = "Hello";
char dest[20];

// String operations
strcpy(dest, src);                    // Copy string
strncpy(dest, src, sizeof(dest)-1);   // Safe copy
strcat(dest, " World");               // Concatenate
int len = strlen(dest);               // Get length
int cmp = strcmp(str1, str2);         // Compare strings
char *found = strstr(dest, "World");  // Find substring
```

### Safe String Operations
```c
// Safe string copy
char *safe_strcpy(char *dest, const char *src, size_t dest_size) {
    if (dest_size == 0) return dest;
    
    strncpy(dest, src, dest_size - 1);
    dest[dest_size - 1] = '\0';
    return dest;
}

// String duplication with malloc
char *string_duplicate(const char *src) {
    if (src == NULL) return NULL;
    
    size_t len = strlen(src) + 1;
    char *dup = malloc(len);
    if (dup == NULL) return NULL;
    
    return strcpy(dup, src);
}
```

### String Tokenization
```c
#include <string.h>

char text[] = "apple,banana,cherry";
char *token = strtok(text, ",");

while (token != NULL) {
    printf("Token: %s\n", token);
    token = strtok(NULL, ",");
}
```

---

## Arrays

### Static Arrays
```c
// Declaration and initialization
int arr1[5] = {1, 2, 3, 4, 5};
int arr2[] = {1, 2, 3};  // Size inferred
int matrix[3][3] = {{1,2,3}, {4,5,6}, {7,8,9}};

// Array size calculation
#define ARRAY_SIZE(arr) (sizeof(arr) / sizeof((arr)[0]))
```

### Dynamic Arrays
```c
typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} DynamicArray;

DynamicArray* array_create(size_t initial_capacity) {
    DynamicArray *arr = malloc(sizeof(DynamicArray));
    if (!arr) return NULL;
    
    arr->data = malloc(initial_capacity * sizeof(int));
    if (!arr->data) {
        free(arr);
        return NULL;
    }
    
    arr->size = 0;
    arr->capacity = initial_capacity;
    return arr;
}

void array_push(DynamicArray *arr, int value) {
    if (arr->size >= arr->capacity) {
        size_t new_capacity = arr->capacity * 2;
        int *new_data = realloc(arr->data, new_capacity * sizeof(int));
        if (!new_data) return;  // Handle error
        
        arr->data = new_data;
        arr->capacity = new_capacity;
    }
    
    arr->data[arr->size++] = value;
}
```

---

## Hash Tables

### Simple Hash Table Implementation
```c
#define TABLE_SIZE 100

typedef struct Node {
    char *key;
    int value;
    struct Node *next;
} Node;

typedef struct {
    Node *buckets[TABLE_SIZE];
} HashTable;

// Simple hash function
unsigned int hash(const char *key) {
    unsigned int hash_value = 0;
    while (*key) {
        hash_value = (hash_value << 5) + *key++;
    }
    return hash_value % TABLE_SIZE;
}

// Insert key-value pair
void hash_insert(HashTable *table, const char *key, int value) {
    unsigned int index = hash(key);
    Node *new_node = malloc(sizeof(Node));
    if (!new_node) return;
    
    new_node->key = string_duplicate(key);
    new_node->value = value;
    new_node->next = table->buckets[index];
    table->buckets[index] = new_node;
}

// Search for key
int hash_search(HashTable *table, const char *key, int *value) {
    unsigned int index = hash(key);
    Node *current = table->buckets[index];
    
    while (current) {
        if (strcmp(current->key, key) == 0) {
            *value = current->value;
            return 1;  // Found
        }
        current = current->next;
    }
    return 0;  // Not found
}
```

---

## File Operations

### Basic File I/O
```c
#include <stdio.h>

// File opening modes
FILE *fp = fopen("file.txt", "r");   // Read
FILE *fp = fopen("file.txt", "w");   // Write (truncate)
FILE *fp = fopen("file.txt", "a");   // Append
FILE *fp = fopen("file.txt", "r+");  // Read/Write

if (fp == NULL) {
    perror("Error opening file");
    return -1;
}

// Always close files
fclose(fp);
```

### Reading and Writing
```c
// Character I/O
int ch = fgetc(fp);
fputc('A', fp);

// String I/O
char buffer[256];
fgets(buffer, sizeof(buffer), fp);
fputs("Hello\n", fp);

// Formatted I/O
int num;
fscanf(fp, "%d", &num);
fprintf(fp, "Number: %d\n", num);

// Binary I/O
int data[10];
fread(data, sizeof(int), 10, fp);
fwrite(data, sizeof(int), 10, fp);
```

### File Position and Error Handling
```c
// File positioning
fseek(fp, 0, SEEK_SET);    // Beginning
fseek(fp, 0, SEEK_END);    // End
fseek(fp, 10, SEEK_CUR);   // Current + 10

long pos = ftell(fp);      // Get current position
rewind(fp);                // Reset to beginning

// Error checking
if (ferror(fp)) {
    printf("File error occurred\n");
}

if (feof(fp)) {
    printf("End of file reached\n");
}
```

---

## Advanced Memory Concepts

### Memory Alignment
```c
#include <stddef.h>
#include <stdalign.h>

// Structure padding
struct Example {
    char c;    // 1 byte
    // 3 bytes padding
    int i;     // 4 bytes
    char c2;   // 1 byte
    // 3 bytes padding
};

// Packed structures (compiler-specific)
struct __attribute__((packed)) PackedExample {
    char c;
    int i;
    char c2;
};

// Alignment control
_Alignas(16) char aligned_buffer[256];
```

### Memory Pools
```c
typedef struct {
    void *memory;
    size_t block_size;
    size_t num_blocks;
    void *free_list;
} MemoryPool;

MemoryPool* pool_create(size_t block_size, size_t num_blocks) {
    MemoryPool *pool = malloc(sizeof(MemoryPool));
    if (!pool) return NULL;
    
    pool->memory = malloc(block_size * num_blocks);
    if (!pool->memory) {
        free(pool);
        return NULL;
    }
    
    pool->block_size = block_size;
    pool->num_blocks = num_blocks;
    
    // Initialize free list
    char *ptr = (char*)pool->memory;
    pool->free_list = ptr;
    
    for (size_t i = 0; i < num_blocks - 1; i++) {
        *(void**)ptr = ptr + block_size;
        ptr += block_size;
    }
    *(void**)ptr = NULL;
    
    return pool;
}
```

### Custom Allocators
```c
// Arena allocator
typedef struct {
    char *memory;
    size_t size;
    size_t offset;
} Arena;

void* arena_alloc(Arena *arena, size_t size) {
    if (arena->offset + size > arena->size) {
        return NULL;  // Out of memory
    }
    
    void *ptr = arena->memory + arena->offset;
    arena->offset += size;
    return ptr;
}

void arena_reset(Arena *arena) {
    arena->offset = 0;  // Reset all allocations
}
```

---

## Theoretical Foundations

### Complexity Analysis
```c
// O(1) - Constant time
int get_first_element(int *arr) {
    return arr[0];
}

// O(n) - Linear time
int linear_search(int *arr, int size, int target) {
    for (int i = 0; i < size; i++) {
        if (arr[i] == target) return i;
    }
    return -1;
}

// O(log n) - Logarithmic time
int binary_search(int *arr, int size, int target) {
    int left = 0, right = size - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

// O(n²) - Quadratic time
void bubble_sort(int *arr, int size) {
    for (int i = 0; i < size - 1; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(&arr[j], &arr[j + 1], sizeof(int));
            }
        }
    }
}
```

### Data Structure Design Principles

#### Abstract Data Types (ADTs)
```c
// Stack ADT
typedef struct {
    int *data;
    int top;
    int capacity;
} Stack;

// Interface functions
Stack* stack_create(int capacity);
void stack_push(Stack *s, int value);
int stack_pop(Stack *s);
int stack_peek(Stack *s);
bool stack_is_empty(Stack *s);
void stack_destroy(Stack *s);
```

#### Memory Layout Considerations
```c
// Cache-friendly structure (Array of Structures)
typedef struct {
    float x, y, z;  // Position
    float r, g, b;  // Color
} Vertex_AoS;

// SIMD-friendly structure (Structure of Arrays)
typedef struct {
    float *x, *y, *z;  // Positions
    float *r, *g, *b;  // Colors
    size_t count;
} Vertex_SoA;
```

### Algorithm Design Patterns

#### Divide and Conquer
```c
void merge_sort(int *arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        
        merge_sort(arr, left, mid);
        merge_sort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}
```

#### Dynamic Programming
```c
// Memoization example: Fibonacci
int fib_memo(int n, int *memo) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo);
    return memo[n];
}
```

### System-Level Concepts

#### Memory Hierarchy
- **Registers**: Fastest, smallest (CPU registers)
- **L1 Cache**: ~1-2 cycles, ~32KB
- **L2 Cache**: ~10 cycles, ~256KB
- **L3 Cache**: ~40 cycles, ~8MB
- **RAM**: ~100+ cycles, GBs
- **Storage**: ~10,000+ cycles, TBs

#### Virtual Memory
```c
// Memory mapping
#include <sys/mman.h>

void *mapped = mmap(NULL, size, PROT_READ | PROT_WRITE,
                    MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
if (mapped == MAP_FAILED) {
    perror("mmap failed");
}

// Remember to unmap
munmap(mapped, size);
```

---

## Best Practices Summary

1. **Always check return values** from malloc, fopen, etc.
2. **Free all allocated memory** and set pointers to NULL
3. **Use const correctness** for read-only parameters
4. **Validate input parameters** in functions
5. **Use static analysis tools** like Valgrind, AddressSanitizer
6. **Follow consistent naming conventions**
7. **Document complex algorithms** and data structures
8. **Consider cache locality** in data structure design
9. **Profile before optimizing** - measure performance bottlenecks
10. **Write defensive code** - assume inputs can be malicious

---

*This cheatsheet covers fundamental to advanced C programming concepts. Practice implementing these patterns and always prioritize code safety and clarity.*

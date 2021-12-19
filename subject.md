# Subject: Dynamic Vector

Let's create a dynamic vector data-structure in C.

To understand more about what we are about to implement check out:

https://doc.rust-lang.org/rust-by-example/std/vec.html

https://www.cplusplus.com/reference/vector/vector/

You can use the tests in `utests.c` to test your own implementation.
Correct implementations (but not the only solutions!) can be found
in vec.c. However it's better if you remove that and try yourself
first!

## Struct

We create a struct called `s_vec` and typedef it to `t_vec`.

```c

typedef struct s_vec
{
    uint8_t *memory;    // Pointer to the first byte of allocated memory.
    size_t  elem_size;  // Size of a vector element in bytes.
    size_t  alloc_size; // Total size of allocated bytes.
    size_t  len;        // Length of the active part of the vector assert
                        // `elem_size` chunks.
}   t_vec;

```

When we access elements in the vector our bounds are 0 -> len. We might have
allocated more memory in total, but we will only access memory in the byte-range
0 -> len * elem_size.

## Implementation

Here is our `vec.h` header file with implementation prototypes;

```c

#ifndef VEC_H
# define VEC_H

#include "stdlib.h"
#include "stdint.h"
#include "unistd.h"
#include "string.h"
#include "stdbool.h"

typedef struct s_vec
{
    uint8_t *memory;
    size_t  elem_size;
    size_t  alloc_size;
    size_t  len;
}   t_vec;

ssize_t vec_new(t_vec *src, size_t len, size_t elem_size);
void    vec_free(t_vec *src);
ssize_t vec_from(t_vec *dst, void *src, size_t len, size_t elem_size);
ssize_t vec_push(t_vec *src, void *elem);
ssize_t vec_pop(void *dst, t_vec *src);
ssize_t vec_copy(t_vec *dst, t_vec *src);
void    *vec_get(t_vec *src, size_t index);
ssize_t vec_insert(t_vec *dst, void *elem, size_t index);
ssize_t vec_remove(t_vec *src, size_t index);
ssize_t vec_append(t_vec *dst, t_vec *src);
ssize_t vec_prepend(t_vec *dst, t_vec *src);
void    vec_iter(t_vec *src, void (*f) (void *));
void    vec_map(t_vec *dst, t_vec *src, void (*f) (void *));
void    vec_filter(t_vec *dst, t_vec *src, bool (*f) (void *));
void    vec_reduce(void *dst, t_vec *src, void (*f) (void *, void *));

#endif

```

## Ex0: vec_new

Create a function `vec_new` which will take a pinter to a `t_vec` and
allocate len * elem_size amount of bytes in the buffer as well as
initialize it's length and element size.

```c

ssize_t vec_new(t_vec *dst, size_t init_alloc, size_t elem_size);

int main(void)
{
    t_vec t1;

    assert(vec_new(&t1, 0, 0) == -1);
    assert(vec_new(&t1, 0, 1) == -1);
    assert(vec_new(&t1, 0, 1) == -1);
    assert(vec_new(&t1, 10, 1) == 10);
    vec_free(&t1);
}

```

## Ex1: vec_free

Create a function `vec_free` which free's the allocated resources
in `src` and zeroes it's fields.

```c

void vec_free(t_vec *src);

int main(void)
{
    t_vec t1;

    assert(vec_new(&t1, 10, 1) == 10);
    vec_free(&t1);
    assert(t1.len == 0);
    assert(t1.alloc_size == 0);
    assert(t1.elem_size == 0);
    assert(t1.memory == NULL);
}

```
## Ex2: vec_from

Create a function `vec_from` which takes in a pointer to some memory,
which then will be copied over to the new vector.

```c

ssize_t vec_from(t_vec *dst, void *src, size_t len, size_t elem_size);

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    assert(memcmp(t1.memory, base, sizeof(base)) == 0);
    vec_free(&t1);
}

```

## Ex3: vec_copy

Create a function `vec_copy`. The copy function is very simple and
will only copy as many bytes as are available in the `dst` vector.

```c

ssize_t vec_copy(t_vec *dst, t_vec *src);

int main(void)
{
    t_vec   t1;
    t_vec   t2;
    int     base[] = {1, 2, 3, 4, 5};

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    assert(vec_new(&t2, 5, sizeof(int)) > 0);
    assert(vec_copy(&t2, &t1) > 0);
    assert(memcmp(t2.memory, base, sizeof(base)) == 0);
    vec_free(&t1);
    vec_free(&t2);
}

```

## Ex4: vec_resize

Create a function `vec_resize` which will take in a `target_size` parameter and
either srink (destructively) or grow the vector to the target size copying
the old contents over to the new alloaction.

```c

static ssize_t vec_resize(t_vec *src, size_t target_size);

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    assert(vec_resize(&t1, 100) == 100);
    assert(memcmp(t1.memory, base, sizeof(base)) == 0);
    vec_free(&t1);
}

```

## Ex5: vec_push

Create a function `vec_push` which takes in a vector and a pointer to an
element to be pushed to the end of the vector.

```c

ssize_t vec_push(t_vec *dst, void *src);

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};
    int     expect[] = {2, 4};

    assert(vec_new(&t1, 1, sizeof(int)) > 0);
    vec_push(&t1, &base[1]);
    vec_push(&t1, &base[3]);
    assert(memcmp(t1.memory, expect, sizeof(expect)) == 0);
    vec_free(&t1);
}

```

## Ex6: vec_pop

Create a function `vec_pop` which will remove the last element from
the vector and copy it to `dst`.

```c

ssize_t vec_pop(void *dst, t_vec *src);

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};
    int     expect = 0;

    assert(vec_new(&t1, 1, sizeof(int)) > 0);
    vec_push(&t1, &base[1]);
    vec_push(&t1, &base[3]);
    vec_pop(&expect, &t1);
    assert(expect == 4);
    vec_pop(&expect, &t1);
    assert(expect == 2);
    assert(t1.len == 0);
    vec_free(&t1);
    printf("test_vec_pop successful!\n");
}

```

## Ex7: vec_get

Create a function `vec_get` which returns an opaque handle to
the element at `index`.

```c

void *vec_get(t_vec *src, size_t index);

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};
    int     *expect;

    assert(vec_new(&t1, 1, sizeof(int)) > 0);
    vec_push(&t1, &base[1]);
    vec_push(&t1, &base[3]);
    expect = vec_get(&t1, 0);
    assert(*expect == 2);
    expect = vec_get(&t1, 1);
    assert(*expect == 4);
    assert(t1.len == 2);
    vec_free(&t1);
}

```

## Ex8: vec_insert

Create a function `vec_insert` which will insert a new element at
any position in the vector without overwriting existing elements.

```c

ssize_t vec_insert(t_vec *dst, void *src, size_t index);

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};
    int     insert[] = {42, 666, 7};
    int     expect[] = {1, 42, 2, 3, 666, 4, 5, 7};

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    vec_insert(&t1, &insert[0], 1);
    vec_insert(&t1, &insert[1], 4);
    vec_insert(&t1, &insert[2], 7);
    assert(memcmp(t1.memory, expect, sizeof(expect)) == 0);
    vec_free(&t1);
}

```

## Ex9: vec_remove

Create a function `vec_remove` which will remove an element from
any position in the vector without overwriting existing elements.

```c

ssize_t vec_remove(t_vec *src, size_t index);

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};
    int     insert[] = {42, 666, 7};

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    vec_insert(&t1, &insert[0], 1);
    vec_insert(&t1, &insert[1], 4);
    vec_insert(&t1, &insert[2], 7);
    vec_remove(&t1, 1);
    vec_remove(&t1, 3);
    vec_remove(&t1, 5);
    assert(memcmp(t1.memory, base, sizeof(base)) == 0);
    vec_free(&t1);
    printf("test_vec_remove successful!\n");
}

```

## Ex10: vec_append

Create a function `vec_append` which appends vector `src` to `dst`.

```c

ssize_t vec_append(t_vec *dst, t_vec *src);

int main(void)
{
    t_vec   t1;
    t_vec   t2;
    int     base1[] = {1, 2, 3};
    int     base2[] = {4, 5, 6};
    int     expect[] = {1, 2, 3, 4, 5, 6};

    assert(vec_from(&t1, base1, 3, sizeof(int)) > 0);
    assert(vec_from(&t2, base2, 3, sizeof(int)) > 0);
    assert(vec_append(&t1, &t2) > 0);
    assert(memcmp(t1.memory, expect, sizeof(expect)) == 0);
    vec_free(&t1);
    vec_free(&t2);
    printf("test_vec_append successful!\n");
}

```

## Ex11: vec_prepend

Create a function `vec_prepend` which prepends vector `src` to `dst`.

```c

ssize_t vec_prepend(t_vec *dst, t_vec *src);

int main(void)
{
    t_vec   t1;
    t_vec   t2;
    int     base1[] = {1, 2, 3};
    int     base2[] = {4, 5, 6};
    int     expect[] = {4, 5, 6, 1, 2, 3};

    assert(vec_from(&t1, base1, 3, sizeof(int)) > 0);
    assert(vec_from(&t2, base2, 3, sizeof(int)) > 0);
    assert(vec_prepend(&t1, &t2) > 0);
    assert(memcmp(t1.memory, expect, sizeof(expect)) == 0);
    vec_free(&t1);
    vec_free(&t2);
    printf("test_vec_prepend successful!\n");
}

```

## Ex12: vec_iter

Create a function `vec_iter` which takes as an argument
a function `f` applied to each element in the vector.

```c

void vec_iter(t_vec *src, void (*f) (void *));

void iter_tester(void *src)
{
    *(int *)src += 1;
}

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};
    int     expect[] = {2, 3, 4, 5, 6};

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    vec_iter(&t1, iter_tester);
    assert(memcmp(t1.memory, expect, sizeof(expect)) == 0);
    vec_free(&t1);
}

```

## Ex13: vec_map

Create a function `vec_map` which takes as an argument
a function `f` applied to a copy of each element in the vector.
The copied element will be added to vector `dst`.

```c

void vec_map(t_vec *dst, t_vec *src, void (*f) (void *));

void map_tester(void *src)
{
    *(int *)src += 1;
}

int main(void)
{
    t_vec   t1;
    t_vec   t2;
    int     base[] = {1, 2, 3, 4, 5};
    int     expect[] = {2, 3, 4, 5, 6};

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    assert(vec_new(&t2, 5, sizeof(int)) > 0);
    vec_map(&t2, &t1, map_tester);
    assert(memcmp(t2.memory, expect, sizeof(expect)) == 0);
    vec_free(&t1);
    vec_free(&t2);
}

```

## Ex14: vec_filter

Create a function `vec_filter` which takes as an argument
a function `f` applied to a copy of each element in the vector.
The copied element will be added to vector `dst` if `true` is
returned from `f`.

```c

void vec_filter(t_vec *dst, t_vec *src, bool (*f) (void *));

bool filter_tester(void *src)
{
    if (*(int *)src % 2 == 0)
        return (true);
    return (false);
}

int main(void)
{
    t_vec   t1;
    t_vec   t2;
    int     base[] = {1, 2, 3, 4, 5};
    int     expect[] = {2, 4};

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    assert(vec_new(&t2, 5, sizeof(int)) > 0);
    vec_filter(&t2, &t1, filter_tester);
    assert(memcmp(t2.memory, expect, sizeof(expect)) == 0);
    vec_free(&t1);
    vec_free(&t2);
}

```

## Ex15: vec_reduce

Create a function `vec_reduce` which takes as an argument
a function `f` applied to each element in the vector.
Function `f` takes `dst` as it's first argument thus
we can reduce the elements in the vector into one element.

```c

void vec_reduce(void *dst, t_vec *src, void (*f) (void *, void *));

void reduce_tester(void *dst, void *src)
{
    *(int *)dst += *(int *)src;
}

int main(void)
{
    t_vec   t1;
    int     base[] = {1, 2, 3, 4, 5};
    int     result = 0;

    assert(vec_from(&t1, base, 5, sizeof(int)) > 0);
    vec_reduce(&result, &t1, reduce_tester);
    assert(result == 15);
    vec_free(&t1);
    printf("test_vec_reduce successful!\n");
}

```
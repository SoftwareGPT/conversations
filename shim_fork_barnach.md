_&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;"To fork, or not to fork, that is the question."_

---

### Introduction to Shim and Dispatch in C

This document outlines a practical example of implementing the "shim and dispatch" approach in C for managing alternative interface implementations.

### Basic Structure

The foundational code setup involves defining an interface and an initial implementation in separate header and source files.

#### `foo.h`

```c
typedef struct interface_s {
    void (*foo)(void);
} interface_t;

extern interface_t implementation;
```

#### `foo.c`

```c
#include "foo.h"

void foo(void) {}

interface_t implementation = {.foo = foo};
```

### Implementing Shim and Dispatch

To extend our example with alternative implementations, we employ the shim and dispatch technique. 
This involves selecting an implementation at runtime based on some criteria, such as an experiment 
identifier or configuration setting.

#### Modified `foo.c`

```c
#include "foo.h"
#include <stdint.h>

extern void foo_0(void);
extern void foo_1(void);
extern void foo_2(void);

int32_t foo_experiment = 0; // Experiment identifier
```

---

### Alternative Interface Implementations

We extend our example with the "shim and dispatch" approach, enabling the dynamic 
selection of implementation at runtime based on an experiment identifier or configuration.

#### Enhanced `foo.c`

```c
#include "foo.h"
#include <stdint.h>

extern void foo_0(void);
extern void foo_1(void);
extern void foo_2(void);

int32_t foo_experiment = 0; // Dynamically adjustable

void foo(void) {
    switch(foo_experiment) {
        case 0: foo_0(); break;
        case 1: foo_1(); break;
        case 2: foo_2(); break;
        default:
            // Handle unexpected values, e.g., no-op or error handling
            break;
    }
}

interface_t implementation = {.foo = foo};
```

#### Alternative Implementations `foo_0.c`, `foo_1.c`, `foo_2.c`

```c
// foo_0.c
#include "foo.h"
void foo_0(void) { /* Implementation for experiment 0 */ }

// foo_1.c
#include "foo.h"
void foo_1(void) { /* Implementation for experiment 1 */ }

// Repeat for other implementations as needed.
```

This strategy allows for the flexible testing of different code versions, 
supporting dynamic experiment selection at runtime, which can be sourced from 
either web-based or local configurations on the fly. 

It enables parallel execution of multiple versions, aiding in comparative 
analysis and decision-making processes like result voting or fallback strategies.

### Forking vs. Branching Workflow

Forking and branching are two pivotal strategies in version control 
systems that facilitate parallel development tracks. While forking provides 
an isolated copy of the repository for experimentation and external contributions, 
branching allows for divergent development within the same repository.

**Forking Workflow:**

1. **Fork Repository:** Create a personal copy of the repository.
2. **Clone Fork:** Work locally on your machine or in the cloud.
3. **Make changes in your fork:** free to experiment w/o affecting the origin.
4. **Commit and Push:** Update your fork with your changes.
5. **Pull Request:** Propose changes to the original repository.

**Branching Workflow:**

1. **Clone Original Repository:** Start with the main codebase.
2. **Create Feature Branch:** Diverge from the main branch.
3. **Commit and Push Changes:** Regularly update the feature branch.
4. **Pull Request:** Merge changes back into the main branch.

### Conclusion

Forking offers significant advantages in terms of isolation, security, and flexibility, 
especially in open-source and large-scale projects. It allows for independent 
experimentation without risking the stability of the main codebase. 

Importantly, forking also ensures that all heavy data used in experiments, 
external binary dependencies on libraries, and data utilized solely during 
experimentation but not in the final code, do not clutter the repository 
of the origin. This approach maintains the original repository's lightweight 
nature, cleanliness, and minimal or no dependencies, promoting a "branchless" 
development environment. However, it necessitates proactive synchronization 
efforts to minimize integration issues. 

This method is particularly favorable for external contributions or projects 
with strict integration policies, as it upholds the integrity and efficiency 
of the primary codebase while allowing for extensive testing and innovation 
in a separate, dedicated space.

For straightforward scenarios, employing shims for alternative implementations 
of interfaces can be a significantly more efficient approach. This strategy not 
only preserves the integrity and efficiency of the primary codebase while 
allowing for extensive testing and innovation in a separate, dedicated space 
but also upholds the lightweight and clean nature of the original repository 
by avoiding the clutter of experimental data and external dependencies.

_&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;"Humans are stupid. I’m ashamed to be human." Kurt Cobain_

===Further reading

https://trunkbaseddevelopment.com/

https://learn.microsoft.com/en-us/devops/develop/how-microsoft-develops-devops



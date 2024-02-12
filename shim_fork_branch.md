_&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
"To fork, or not to fork, that is the question."_

---

### Introduction to Shim / Dispatch in C

This document outlines a practical example of implementing the "shim and dispatch" 
approach in C for managing alternative interface implementations.

### Basic Structure

The foundational code setup involves defining an interface and an initial 
implementation in separate header and source files.

#### `snafu.h`

```c
typedef struct snafu_if {
    void (*foo)(void);
    void (*bar)(void);
} snafu_if;

extern snafu_if snafu;
```

#### `snafu.c`

```c
#include "snafu.h"

static void foo(void) {} // allows short "verb" names for methods
static void bar(void) {} // without leaking them into global namespace

snafu_if snafu = { .foo = foo, .bar = bar };
```

### Example of making use of the interface:

#### `main.c`

```c
#include "snafu.h"

int main(int argv, const char* argv[]) {
    snafu.foo(); // performance hit: one extra memory/cache access cycle
    snafu.bar(); // hard to impossible to inline for the compiler/linker
    return 0;
}

```

### Implementing Shim / Dispatch

> "In software engineering, a shim is a small piece of code that 
acts as a bridge between two components or layers of a software system. 
Additionally, we can also refer to it as an adapter or a wrapper."

To extend our example with alternative implementations, we employ 
the shim and dispatch technique. 

This involves selecting an implementation at runtime based on some 
criteria, such as an experiment 
identifier or configuration setting.

#### Enhanced `snafu.c`

```c
#include "snafu.h"
#include <stdint.h>

int32_t snafu_ix; // implementation index [0..2]

extern snafu_if snafu_0;
extern snafu_if snafu_1;
extern snafu_if snafu_2; // can be extended to [0..n]

static snafu_if* snafus[3] = {&snafu_0, &snafu_1, &snafu_2};

static void foo(void) { snafus[snafu_ix]->foo(); }
static void bar(void) { snafus[snafu_ix]->bar(); }

snafu_t snafu = { .foo = foo, .bar = bar };
```

In debug/development code:

```assert(0 <= snafu_ix && snafu_ix < countof(snafus));```

in release/production code:

```fatal_if_not(0 <= snafu_ix && snafu_ix < countof(snafus));```

are recommended for range check safety in all dispatcher methods.


### Alternative Implementations `snafu_0.c`, `snafu_1.c`, `snafu_2.c`


#### Alternative `snafu_0.c`

```c
#include "snafu.h"

/* Implementation for the experiment 0 */

static void foo(void) { /* ... */ }
static void bar(void) { /* ... */ }

snafu_if snafu_0 = { .foo = foo, .bar = bar };
```

"Rinse and repeat" for ```snafu_1.c```, ```snafu_2.c``` implementations.  

This strategy allows for the flexible testing of different code versions, 
supporting dynamic experiment selection at runtime, which can be sourced from 
either web-based or local configurations on the fly. 

It can be extended to enable concurrent or serial execution of multiple 
implementations (assuming that implementations are well designed and 
do not have conflicting global state side effects).
Concurrent or serial execution of implementation can be used for aiding 
in comparative  analysis and decision-making processes like result 
voting or algorithms fallback strategies.

When shim/dispatch is not enough for alternative implementations the source code
version control can be used instead or in combination with it. Examples
may include but not limited to:

* Bringing external code and or data into the alternative 
implementation experiments that are useful only at the time of development.
* Prolonged development cycle spawning sever project releases. 
* Code implementation by independent remote developers and maintainers.


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
5. **Delete Feature Branch:** delete feature branch to avoid long 
living out of sync branches and dangers of their accidental merge.


### Advantages of forking


_&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;
"Use the fork, Luke, use the fork..." Yoda_


Forking offers significant advantages over branching in terms of isolation, 
security, and flexibility, especially in open-source and large-scale projects. 
It allows for independent experimentation without risking the stability 
of the main codebase. 

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

### Conclusion

For straightforward scenarios, employing shims for alternative implementations 
of interfaces can be a significantly more efficient approach. This strategy not 
only preserves the integrity and efficiency of the primary codebase while 
allowing for extensive testing and innovation in a separate, dedicated space 
but also upholds the lightweight and clean nature of the original repository 
by avoiding the clutter of experimental data and external dependencies.

### Further reading

https://trunkbaseddevelopment.com/

https://learn.microsoft.com/en-us/devops/develop/how-microsoft-develops-devops



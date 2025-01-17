
<div align="center">
  <img src="https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/src_docs/src/assets/logo_name_300.png">
</div>

BRKGA-MP-IPR - C++ version 2.0
================================================================================

<table>
<tr>
  <td>Documentation</td>
  <td>
    <a href="https://ceandrade.github.io/brkga_mp_ipr_cpp">
    <img src="https://img.shields.io/badge/Tutorial-API-blue.svg" alt="Documentation" />
    </a>
  </td>
</tr>
<tr>
  <td>License</td>
  <td>
    <a href="https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/LICENSE.md">
    <img src="https://img.shields.io/badge/license-BSD--like-blue" alt="License" />
    </a>
    <a href="https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/LICENSE.md">
    <img src="https://img.shields.io/badge/license-MIT-blue" alt="License" />
    </a>
  </td>
</tr>
</table>

Table of Contents
--------------------------------------------------------------------------------

* :speech_balloon: [Introduction](#speech_balloon-introduction)
* :high_brightness: [What is new on version 2.0](#high_brightness-what-is-new-on-version-2.0)
	* [Multiples objectives (Lexicographically)](#multiples-objectives-lexicographically)
	* [Multi-thread mating](#multi-thread-mating)
	* [API changes](#api-changes)
	* [Bug fixes](#bug-fixes)
* :computer: [Installation](#computer-installation)
* :zap: [Usage](#zap-usage---tldr-single-objective)
	* [TL;DR, Single objective](#zap-usage---tldr-single-objective)
	* [TL;DR, Multi objective](#zap-usage---tldr-multi-objective)
* :books: [Tutorial and full documentation](#books-tutorial-and-full-documentation)
* :black_nib: [License and Citing](#black_nib-license-and-citing)
* :construction_worker: [TODO](#construction_worker-todo)
* :pencil2: [Contributing](#pencil2-contributing)

:speech_balloon: Introduction
--------------------------------------------------------------------------------

BRKGA-MP-IPR provides a _very easy-to-use_ framework for the
Multi-Parent Biased Random-Key Genetic Algorithm with Implict Path Relink
(**BRKGA-MP-IPR**). Assuming that your have a _decoder_ to your problem,
we can setup, run, and extract the value of the best solution in less than
5 commands (obvisiously, you may need few other lines fo code to do a proper
test).

We provide a very thorough and easy-to-follow tutorial on using
BRKGA effectively. We strongly recommend you read the tutorial to make
your code works at best within BRKGA. Check out the tutorial and API
documentation: https://ceandrade.github.io/brkga_mp_ipr_cpp

This C++ version provides a fast prototyping API using C++17 standards and
libraries. All code was developed as a header-only library, and have no
external dependencies other than those included in the package. So, you just
need to copy/check out the files and point your compiler's header path to
BRKGA-MP-IPR folder (`-I` on G++ and CLANG++).

This framework can use multiple threads of modern CPUs, by setting a single
parameter (assuming that your decoder is thread-safe). This leverage the
parallel decoding nature that BRKGAs offer, compared to other (meta)
heuristic frameworks.

The code can be compiled using [> GCC 7.2](https://gcc.gnu.org) and [> Clang
6.0](https://clang.llvm.org), and it is very probable that it can be compiled
using other modern C++ compilers, such as Intel and Microsoft compilers. To
use multiple threads, your compiler must support
[OpenMP](https://www.openmp.org). The current version has been long developed,
and it is a very mature code used in production in several companies.
However, it lacks of a proper unit and coverage testing. Such tests are in
the [TODO](#construction_worker-todo) list.

If C++ is not suitable to you, we may find useful the [**Julia
version**](https://github.com/ceandrade/brkga_mp_ipr_julia) of this framework.
We are also developing a
[**Python version**](https://github.com/ceandrade/brkga_mp_ipr_python)
which is in its earlier stages.
However, both Julia and Python versions only support single-objective optimization
at this moment. We have no timeline to implement multi-objective optimization
in such platforms. Also, at this moment, we have no plans to implement
the BRKGA-MP-IPR in other
languages such as Java or C#. But if you want to do so, you are
must welcome. But please, keep the API as close as possible to the C++ API
(or Julia API in case you decide go C), and use the best coding and
documentation practices of your chosen language/framework.

- [**Julia version**](https://github.com/ceandrade/brkga_mp_ipr_julia)
- [**Python version**](https://github.com/ceandrade/brkga_mp_ipr_python)

If you are not familiar with how BRKGA works, take a look on
[Standard BRKGA](http://dx.doi.org/10.1007/s10732-010-9143-1) and
[Multi-Parent BRKGA](http://dx.doi.org/xxx).

:high_brightness: What is new on version 2.0
--------------------------------------------------------------------------------

### Multiples objectives (Lexicographically)

In version 2.0, BRKGA-MP-IPR also deals with multiple objectives in a
lexicographical or priority dominance order. Differing from classical
non-dominance order (using Pareto frontiers), the lexicographical order defines
a _strict preference order among the objective functions._ This leads us to a
partial ordering of the values of the solutions (composed of several values,
each one from one objective function). So, we have the following definition
(abusing a little bit of notation).

| :memo: Definition          |
|:---------------------------|
| Let `A = (f_1, f_2, ..., f_n)` and `A' = (f'_1, f'_2, .., f'_n)` be two vectors for `n` functions `f_1, f_2, ..., f_n`. `A` is lexicographical smaller than `A'`, i.e.,`A < A'` if and only if `f_1 < f'_1`, or `f_1 = f'_1` and `f_2 < f'_2`, or `..., f_1 = f'_1, ..., f_{n-1} = f'_{n-1}` and `f_n < f'_n`.|

For instance, let's assume we have three minimizing objective functions and
four solutions described in the following table:

|Solution|`f_1`      |`f_2`      |`f_3`
|--------|-----------|-----------|----------|
|A       |50         |30         |30
|B       |30         |55         |40
|C       |30         |20         |50
|D       |30         |20         |25

Note that Solution B is better than Solution A because `f_1(A) < f_1(B),`
even though A has much better values for `f_2` and `f_3`. Now,
Solution C is better B because, although `f_1(B) = f_1(C),` we have that
`f_2(B) < f_2(C),` regardless of the value of `f_3.` Solution D
has the best value for all objective functions. Therefore `D < C < B <
A.`

| :warning: Warning          |
|:---------------------------|
| If you really want an algorithm to produce a **non-dominated set of solutions (Pareto frontier)**, this is **NOT** the right algorithm for you. We recommend taking a look at the [NSGA-II](https://doi.org/10.1109/4235.996017) and [MOAB](https://en.wikipedia.org/wiki/MOEA_Framework).

### Multi-thread mating

One of the nice additions to BRKGA-MP-IPR 2.0 is the capability of performing
the mating in parallel. Such capability speeds up the algorithm substantially,
mainly for large chromosomes and large populations. However, when performing
parallel mating, we have some points regarding reproducibility described in Section
[Multi-thread mating](https://ceandrade.github.io/brkga_mp_ipr_cpp/page_guide.html#multi-threading)
in the [tutorial](https://ceandrade.github.io/brkga_mp_ipr_cpp).

### API changes

#### New type `fitness_t`

Due to the inclusion of multi-objective optimization capabilities, the user now must
define a type `fitness_t`, and his/her decoder must return such a type.
The user can do things like this:

Please, check for details in Sections
["TL;DR - Multi objective"](http://localhost:8000/page_guide.html#tl-dr-multi-objective)
and
["Using BRKGA-MP-IPR on multi-objective mode"](https://ceandrade.github.io/brkga_mp_ipr_cpp/page_guide.html#using-brkga-mp-ipr-on-multi-objective-mode)
from the [tutorial](https://ceandrade.github.io/brkga_mp_ipr_cpp).

#### New `BRKGA::setInitialPopulation()`

In the previous versions, `setInitialPopulation()` fills up only the first
population, discarding additional warm solutions if the population is full.
Now, `setInitialPopulation()` fills up all populations.
[More details here.](https://ceandrade.github.io/brkga_mp_ipr_cpp/class_BRKGA_BRKGA_MP_IPR.html#doxid-class-b-r-k-g-a-1-1-b-r-k-g-a-m-p-i-p-r-1a59b05650ede92f5e0107ab606ff6e8b7)

#### Changes in `BRKGA::injectChromosome()`

`injectChromosome()` does not accept initial fitness value anymore. Now, `injectChromosome()` triggers the decoder in all cases (and therefore, time must be measured on its call).
[More details here.](https://ceandrade.github.io/brkga_mp_ipr_cpp/class_BRKGA_BRKGA_MP_IPR.html#doxid-class-b-r-k-g-a-1-1-b-r-k-g-a-m-p-i-p-r-1a0347f67b59bfe36856d1c27c95d4b151)

#### Pump to C++ 17

BRKGA-MP-IPR now uses some features of [C++17 standards](https://en.wikipedia.org/wiki/C%2B%2B17).
Therefore, you must change your building tools to support that.

### Bug fixes

- `BrkgaParams::mutants_percentage` should be of type `double` not `unsigned` ([issue #1]( https://github.com/ceandrade/brkga_mp_ipr_cpp/issues/1)). Thanks @sohaibafifi.
- Fix the shorter method call for `BRKGA_MP_IPR::pathRelinking` ([pull #1](https://github.com/ceandrade/brkga_mp_ipr_cpp/pull/2)). Thanks @afkummer.

:computer: Installation
--------------------------------------------------------------------------------

BRKGA-MP-IPR is a header-only framework, which means that you only need to
download the header files and tell your compiler to include the path to where
the files were downloaded.

Quick example (unix): assume we are in an empty folder. So, we copy/clone
BRKGA-IPR-MP first:

    $ git clone https://github.com/ceandrade/brkga_mp_ipr_cpp
    Cloning into 'brkga_mp_ipr_cpp'...
    remote: Enumerating objects: 118, done.
    remote: Counting objects: 100% (118/118), done.
    remote: Compressing objects: 100% (112/112), done.
    remote: Total 118 (delta 24), reused 0 (delta 0)
    Receiving objects: 100% (118/118), 3.50 MiB | 3.66 MiB/s, done.
    Resolving deltas: 100% (24/24), done.

Let's write a `test.cpp` with the following code:

```cpp
#include "brkga_mp_ipr.hpp"
#include <iostream>

int main() {
    std::cout << "Testing sense: " << BRKGA::Sense::MINIMIZE;
    return 0;
}
```

Then, let's compile and see it works:

    $ g++ --version
    g++ (MacPorts gcc10 10.3.0_0) 10.3.0

    $ g++ -std=c++17 -Ibrkga_mp_ipr/brkga_mp_ipr test.cpp -o test

    $ ./test
    Testing sense: MINIMIZE

Note the Git clones the whole repository that contains the library code,
documents, and examples. All the examples were built using
[GNU/Make](https://www.gnu.org/software/make/) and
[GCC toolchain](https://gcc.gnu.org). However, the code is standard C++,
and we can quickly adapt it to other toolchains such as
[Clang](https://clang.llvm.org),
[Microsoft](https://visualstudio.microsoft.com), or
[Intel](https://software.intel.com/en-us/c-compilers) toolchains.

:zap: Usage - TL;DR, Single objective
--------------------------------------------------------------------------------

The best way to keep it short is to look in the on examples on
[the git repo.](https://github.com/ceandrade/brkga_mp_ipr_cpp/tree/master/examples)
Let's start solving the traditional single-objective
[Traveling Salesman Problem (TSP)](https://en.wikipedia.org/wiki/Travelling_salesman_problem).
First, we must ensure that
[BRKGA::fitness_t](https://ceandrade.github.io/brkga_mp_ipr_cpp/namespace_BRKGA.html#doxid-namespace-b-r-k-g-a-1ae9551fcbbfd6072b95e5f112e3b1565e)
has the right single-object type. Let's take a look at the trimmed version of
[`fitness_type.hpp`](https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/brkga_mp_ipr/fitness_type.hpp).
This is a trimmed copy:

```cpp
#include <limits>
#include <tuple>

namespace BRKGA {

typedef double fitness_t;

//...
} // end namespace BRKGA_MP_IPR
```

Here, `fitness_t` defines the type of the objective function value. In the vast
majority of the cases, `double` suffices. Let's take a look into the main call
[main_minimal.cpp](https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/examples/tsp/src/single_obj/main_minimal.cpp).
This is a trimmed copy:

```cpp
#include "tsp/tsp_instance.hpp"
#include "decoders/tsp_decoder.hpp"
#include "brkga_mp_ipr.hpp"
#include <iostream>
#include <stdexcept>
#include <string>
using namespace std;

int main(int argc, char* argv[]) {
    if(argc < 4) {
        cerr << "Usage: "<< argv[0]
             << " <seed> <config-file> <num-generations>"
                " <tsp-instance-file>" << endl;
        return 1;
    }

    const unsigned seed = stoi(argv[1]);
    const string config_file = argv[2];
    const unsigned num_generations = stoi(argv[3]);
    const string instance_file = argv[4];

    auto instance = TSP_Instance(instance_file);

    auto [brkga_params, control_params] =
	      BRKGA::readConfiguration(config_file);

    TSP_Decoder decoder(instance);

    BRKGA::BRKGA_MP_IPR<TSP_Decoder> algorithm(
            decoder, BRKGA::Sense::MINIMIZE, seed,
            instance.num_nodes, brkga_params);

    algorithm.initialize();

    algorithm.evolve(num_generations);

    auto best_cost = algorithm.getBestFitness();
    cout << "Best cost: " << best_cost;

    return 0;
}
```

You can identify the following basic steps:

1. Create a data structure to hold your input data. This object should be passed
   to the decoder object/functor (example
   [`tsp/tsp_instance.hpp`](https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/examples/tsp/src/single_obj/tsp/tsp_instance.hpp));

2. Certify that [BRKGA::fitness_t](https://ceandrade.github.io/brkga_mp_ipr_cpp/namespace_BRKGA.html#doxid-namespace-b-r-k-g-a-1ae9551fcbbfd6072b95e5f112e3b1565e)
   has the correct type;

3. Implement a decoder object/functor. This function translates a chromosome
   (array of numbers in the interval [0,1]) to a solution for your problem. The
   decoder must return the solution value or cost to be used as fitness by BRKGA
   (example
   [`decoders/tsp_decoder.hpp`](https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/examples/tsp/src/single_obj/decoders/tsp_decoder.hpp));

4. Load the instance and other relevant data;

5. Read the algorithm parameters using `BRKGA::readConfiguration()`; or
   create a `BRKGA::BrkgaParams` object by hand;

6. Create an `BRKGA::BRKGA_MP_IPR` algorithm object;

7. Call `BRKGA::BRKGA_MP_IPR::initialize()` to init the BRKGA state;

8. Call `BRKGA::BRKGA_MP_IPR::evolve()` to optimize;

9. Call `BRKGA::BRKGA_MP_IPR::getBestFitness()` and/or
   `BRKGA::BRKGA_MP_IPR::getBestChromosome()` to retrieve the best solution.

[`main_minimal.cpp`](https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/examples/tsp/src/single_obj/main_minimal.cpp)
provides a very minimal example to understand the necessary steps to use the
BRKGA-MP-IPR framework. However,
[`main_complete.cpp`](https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/examples/tsp/src/single_obj/main_complete.cpp)
provides a full-featured code, handy for scientific use, such as
experimentation and paper writing. This code allows fine-grained control of
the optimization, shows several features of BRKGA-MP-IPR such as the resets,
chromosome injection, and others. It also logs
all optimization steps, _creating outputs easy to be parsed._ **You should use
this code for serious business and experimentation.**

These are the basic steps, but I do recommend the full reading of [the guide](https://ceandrade.github.io/brkga_mp_ipr_cpp).

:zap: Usage - TL;DR, Multi objective
--------------------------------------------------------------------------------

| :warning: Warning          |
|:---------------------------|
| Again, if you really want an algorithm to produce a **non-dominated set of solutions (Pareto frontier)**, this is **NOT** the right algorithm for you. We recommend taking a look at the [NSGA-II](https://doi.org/10.1109/4235.996017) and [MOAB](https://en.wikipedia.org/wiki/MOEA_Framework).

To use BRKGA-MP-IPR in the multi-objective mode, we first must set
[BRKGA::fitness_t](https://ceandrade.github.io/brkga_mp_ipr_cpp/namespace_BRKGA.html#doxid-namespace-b-r-k-g-a-1ae9551fcbbfd6072b95e5f112e3b1565e)
according to the number of objectives we want. In the
[repo example](https://github.com/ceandrade/brkga_mp_ipr_cpp/tree/master/examples/tsp/src/multi_obj),
we consider the TSP with two objectives: first, we must minimize the total tour
length, and second, the size of the largest edge in the tour. For that, we must
change the file
[fitness_type.hpp](https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/brkga_mp_ipr/fitness_type.hpp)
to reflect such a thing. In this example, we use the standard `std::tuple`:

```cpp
#include <limits>
#include <tuple>

namespace BRKGA {

typedef std::tuple<double, double> fitness_t;

//...
} // end namespace BRKGA_MP_IPR
```

In this case, the first component of the tuple holds the tour length, and the
second contains the largest edge. On Section
[Using BRKGA-MP on multi-objective mode](https://ceandrade.github.io/brkga_mp_ipr_cpp/page_guide.html#using-brkga-mp-ipr-on-multi-objective-mode),
we talk with more details about multi-objective problems. Just keep in mind,
although you could use any type for your `fitness_t`, you should prefer to use
`std::tuple`.

The remaining code is almost identical to the single-objective. The only
differences are in computing the largest edge, and printing such information on
the main call. All the steps described briefly in the
[previous section](#zap-usage---tldr-single-objective) are also used here.

:books: Tutorial and full documentation
--------------------------------------------------------------------------------

We provide a very thorough and easy-to-follow tutorial on using BRKGA effectively.
We strongly recommend you read the tutorial to make your code works at best within BRKGA.

Check out the tutorial and API documentation:
https://ceandrade.github.io/brkga_mp_ipr_cpp

:black_nib: License and Citing
--------------------------------------------------------------------------------

BRKGA-MP-IPR uses a permissive BSD-like license and it can be used as it
pleases you. And since this framework is also part of an academic effort, we
kindly ask you to remember to cite the originating paper of this work.
Indeed, Clause 4 estipulates that "all publications, softwares, or any other
materials mentioning features or use of this software (as a whole package or
any parts of it) and/or the data used to test it must cite the following
article explicitly":

> C.E. Andrade. R.F. Toso, J.F. Gonçalves, M.G.C. Resende. The Multi-Parent
> Biased Random-key Genetic Algorithm with Implicit Path Relinking. _European
> Journal of Operational Research_, volume 289, number 1, pages 17–30, 2021.
> DOI [10.1016/j.ejor.2019.11.037](https://doi.org/10.1016/j.ejor.2019.11.037)

If you are using the multi-objective version, you must also cite this paper:

> C.E. Andrade, L.S. Pessoa, S. Stawiarski. The Physical Cell Identity
> Assignment Problem: a Multi-objective Optimization Approach.
> _IEEE Transactions on Evolutionary Computation_, volume XXX, number X,
> pages XX–XX, 2022.
> DOI: [10.1016/j.ejor.2019.11.037](https://doi.org/10.1016/j.ejor.2019.11.037).

You may also consider to cite the following papers from people that helped
to find bugs and develop new features for BRKGA-MP-IPR 2.0:

> A.F. Kummer N., L.S. Buriol, O.C.B. de Araujo. A biased random key genetic
> algorithm applied to the VRPTW with skill requirements and synchronization
> constraints. *Proceedings of the 2020 Genetic and Evolutionary Computation
> Conference (GECCO '20)*, pages 717-724, 2020.
> DOI: [10.1145/3377930.3390209](https://doi.org/10.1145/3377930.3390209).

You can download all references for this
[:open_file_folder: Bibtex](https://ceandrade.github.io/brkga_mp_ipr_cpp/_downloads/9e03f77296678467b193eb411a9ebc25/references.bib),
or this
[:open_file_folder: RIS](https://ceandrade.github.io/brkga_mp_ipr_cpp/_downloads/89ca66b5c665a51fcb314b01f33d167e/references.ris)
files.

[Check it out the full license.](https://github.com/ceandrade/brkga_mp_ipr_cpp/blob/master/LICENSE.md)

:construction_worker: TODO
--------------------------------------------------------------------------------

CI and tests side:

- Implement unit tests and certify coverage;

- Configure Travis-Ci correctly, such that we can run tests on Mac OSX and
  Windows too.

:pencil2: Contributing
--------------------------------------------------------------------------------

[Contribution guidelines for this project](CONTRIBUTING.md)

# STUDY NOTES

## CMake : `PUBLIC` - `PRIVATE` - `INTERFACE`

by : [_Lei Mao_](https://leimao.github.io/blog/CMake-Public-Private-Interface/)

In CMake, for any `target`, in the preprocessing stage, it comes with a `INCLUDE_DIRECTORIES` and a `INTERFACE_INCLUDE_DIRECTORIES` for searching the header files building. `target_include_directories` will populate all the directories to `INCLUDE_DIRECTORIES` and/or `INTERFACE_INCLUDE_DIRECTORIES` depending on the keyword `<PRIVATE|PUBLIC|INTERFACE>` we specified. The `INCLUDE_DIRECTORIES` will be used for the current target only and the `INTERFACE_INCLUDE_DIRECTORIES` will be appended to the `INCLUDE_DIRECTORIES` of any other target which has dependencies on the current target. With such settings, the configurations of `INCLUDE_DIRECTORIES` and `INTERFACE_INCLUDE_DIRECTORIES` for all building targets are easy to compute and scale up even for multiple hierarchical layers of building dependencies and many building targets

>| Include Inheritance | Description                                                                                                                                                                                                                                          |
>|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
>| `PUBLIC`              | All the directories following `PUBLIC` will be used for the current target and the other targets that have dependencies on the current target, i.e., appending the directories to INCLUDE_DIRECTORIES and `INTERFACE_INCLUDE_DIRECTORIES`.               |
>| `PRIVATE`             | All the include directories following `PRIVATE` will be used for the current target only, i.e., appending the directories to `INCLUDE_DIRECTORIES`.                                                                                                      |
>| `INTERFACE`           | All the include directories following `INTERFACE` will **NOT** be used for the current target but will be accessible for the other targets that have dependencies on the current target, i.e., appending the directories to `INTERFACE_INCLUDE_DIRECTORIES`. |

### Link Inheritance

Similarly, for any `target`, in the linking stage, we would need to decide, given the `item` to be linked, whether we have to put the `item` in the link dependencies, or the link interface, or both, in the compiled `target`. Here the link dependencies means the item has some implementations that the `target` would use, and it is linked to the `item`, so that whenever we call the functions or methods corresponding to those implementations it will always be mapped correctly to the implementations in `item` via the link, whereas the link interface means the `target` becomes an interface for linking the `item` for other `target`s which have dependencies on the `target`, and the `target` does not have to use `item` at all.

>| Link Type |                                                                                   Description                                                                                  |
>|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
>| `PUBLIC`    | All the objects following `PUBLIC` will be used for linking to the current target and providing the interface to the other targets that have dependencies on the current target. |
>| `PRIVATE`   | All the objects following `PRIVATE` will only be used for linking to the current target.                                                                                         |
>| `INTERFACE` | All the objects following `INTERFACE` will only be used for providing the interface to the other targets that have dependencies on the current target.                           |

For example, if the `fruit` library has the implementation of functions, such as `size` and `color`, and the apple library has a function apple_`size` which called the `size` from the `fruit` library and was `PRIVATE` linked with the `fruit` library. We could create an executable `eat_apple` that calls `apple_size` by `PUBLIC` or `PRIVATE` linking with the apple library. However, if we want to create an executable `eat_apple` that calls the `size` and `color` from the `fruit` library, only linking with the `apple` library will cause building error, since the `fruit` library was not part of the interface in the `apple` library, and is thus inaccessible to `eat_apple`. To make the `apple` library to inherit the `size` and `color` from the `fruit` library, we have to make the linking of the `apple` library to the the `fruit` library `PUBLIC` instead of `PRIVATE`.

### Conclusions

The CMake builds a hierarchical project via the include interface or link interface. The “inheritance” mechanism in C++ is built upon the include interface or link interface.
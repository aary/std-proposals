# Structured bindings with polymorphic lambdas

This paper introduces the idea of using structured bindings with polymorphic
lambdas.  This will allow code like the following

```c++
std::for_each(map, [](const auto& [key, value]) {
    cout << key << " " << value << endl;
});
```

Read the full paper [here](https://goo.gl/i9qV3Z)

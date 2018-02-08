# Tightening the constraints on std::function

```c++
#include <iostream>
#include <functional>

const int& foo(std::function<const int&()> bar) {
    return bar();
}

int main() {
    std::cout << foo([] { return 1; }) << std::endl;
}
```

The code above exhibits undefined behavior.  Fix it.

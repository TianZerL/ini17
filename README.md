# ini17
A header-only C++17 ini file parser and generator.

# Compile
Just include it and compile, a compiler that supports C++17 is needed, has been tested on VS2019, g++11 and clang++12.

# Sample
Here show sample usages of cmdline. Suppose we have `test.ini` and `test2.ini`  
test.ini:
```ini
# test.ini

[section1]
key1 = 1
key2 = 1.23
key3 = abc

[section2]
key1 = 2
key2 = 4.56
key3 = def
```
test2.ini:
```ini
; test2.ini with unspecified section

key1 = 1
key2 = 2
key3 = 3
```

Our code:
```C++
#include <iostream>
#include <fstream>
#include <cstdint>

#include "ini17.hpp"

int main()
{
    ini17::Parser parser;
    ini17::Generator generator;

    // load and parse file "test.ini"
    if (parser.parseFile("test.ini"))
    {
        // get "section1", will return a std::optional
        if (auto section1Check = parser.get<ini17::Section>("section1"))
        {
            // get Section object
            auto section1 = section1Check.value();

            // check and get "key1"
            if (auto key1Check = section1.get<int>("key1"))
            {
                auto key1 = key1Check.value();
                std::cout << "key1: " << key1 << std::endl;
            }
            // if you do not pass in a nonexistent key, check is unnecessary
            auto key1 = *section1.get<int>("key1");
            std::cout << "key1: " << key1 << std::endl;

            // or you can use the [] operator, just like a map
            std::cout << "key2: " << static_cast<std::string>(section1["key2"]) << std::endl;
            section1["key2"] = "I was 1.23";
            std::cout << "key2: " << static_cast<std::string>(section1["key2"]) << std::endl;

            // if you put a nonexistent key, it will be inserted
            section1["key4"] = false;

            // push our new section to generator and generate a ini file
            generator.push(std::move(section1));
            generator.generateFile("new.ini");
        }

        // you can also get values directly from the parser object
        auto key2 = *parser.get<double>("section2", "key2");
        std::cout << "key2: " << key2 << std::endl;
    }

    // load and parse file "test2.ini" with a unspecified section
    parser.parseFile("test2.ini");

    // convert to specified type
    auto key1 = *parser.get<int>("key1");
    auto key2 = *parser.get<std::uint64_t>("key2");
    auto key3 = *parser.get<double>("key3");

    std::cout << key1 << " " << key2 << " " << key3 << std::endl;

    // get unspecified section by default section name
    // you can set it by Parser::setDefaultSectionName()
    auto defaultSection = *parser.get<ini17::Section>(parser.getDefaultSectionName());
    // by default, the name of unspecified section will be the value of Parser::getDefaultSectionName(),
    // this will add an section label in generator,
    // if you don't want it, you need to set section name to an empty string
    defaultSection.setName(std::string_view{});
    generator.clear();
    generator.push(defaultSection);
    // generate to a string
    auto result = generator.generate();
    std::cout << "new2.ini:\n"
              << result;

    // check errors
    for (auto &&e : parser.error())
        std::cerr << e << std::endl;

    return 0;
}
```

Execution results:
```shell
$ ./test
key1: 1
key1: 1
key2: 1.23
key2: I was 1.23
key2: 4.56
1 2 3
new2.ini:
key3 = 3
key2 = 2
key1 = 1
```

And `new.ini`:
```ini
[section1]
key4 = 0
key3 = abc
key2 = I was 1.23
key1 = 1
```

# Extra
- The comment line starts with `#` or `;`.   
- Parser allows sections to be repeated and will be merged.   
- Keys between different sections can be repeated, but the repeated keys in the same section will be subject to the first value.

# Table of contents
- [Table of contents](#table-of-contents)
- [Introduction](#introduction)
- [String API](#string-api)
- [Raw Buffer API](#raw-buffer-api)
- [Stream API](#stream-api)
- [Serio::Size](#seriosize)
- [STL Data Types](#stl-data-types)
- [Custom Structures and Classes](#custom-structures-and-classes)
- [Custom Containers](#custom-containers)
- [Raw Arrays](#raw-arrays)
- [Advanced Usage](#advanced-usage)

# Introduction
This library is easy to use. Here is an explanation of the API. ```Serio``` has two type of APIs:

+ Serialization to and deserialization from buffers which is preferred way of using ```Serio``` because this API is very fast.
+ Serialization to and deserialization from streams which is used where the memory is not enough to use the buffer API or you have a special stream that has to be used instead of buffer API.

Functions that serialize take unlimited number of supported data types and serialize them into a string or raw array or stream.

Functions that deserialize take unlimited number of supported data types and deserialize them from a string or raw array or stream.

# String API
This API contains 4 functions ```Serio::serialize()```, ```Serio::deserialize()```, ```Serio::save()``` and ```Serio::load()``` which are explained below.

You can serialize data types into a string of characters (```Serio::ByteArray``` which is std::basic_string<char>). Here is an example of serializing to string:

``` c++
int A = 0, B = 0;
Serio::ByteArray str = Serio::serialize(A, B);
```

You can deserialize data types from a string of characters. Here is an example of deserializing from string:

``` c++
int A, B;
Serio::ByteArray str = "<serialized-data>";
size_t consumed = Serio::deserialize(str, A, B);
```

You can also save the data to be serialized to file directly like this:

``` c++
int A = 0, B = 0;
bool success = Serio::save("<file-path>", A, B);
```

You can also read and deserialize data directly from file like this:

``` c++
int A, B;
bool success = Serio::load("<file-path>", A, B);
```

If you want to so serialize or deserialize multiple times you can just pass all of the data types to the corresponding functions but when it is not possible this is the **wrong** way of doing it:

``` c++
int A = 0;
auto data = Serio::serialize(A);
int B = 0;
data = Serio::serialize(data, B);
```

This is the right way of doing it:

``` c++
int A = 0;
auto data = Serio::serialize(A);
int B = 0;
data += Serio::serialize(B);
```

But the addition makes the code slower so pass all the types at once as much as you can.

# Raw Buffer API
This API contains 3 functions ```Serio::size()```, ```Serio::fill()``` and ```Serio::deserialize()``` which are explained below.

You can serialize data types into a char sequence. Here is an example of serializing to raw buffer:

``` c++
int A = 0, B = 0;
auto data = new char[Serio::size(A, B)];
Serio::fill(data, A, B);
```

You can set the buffer size large enough to not be concerned about overflow or you can know the exact size by using ```Serio::size()``` function.

You can deserialize data types from a char sequence. Here is an example of deserializing from raw buffer:

``` c++
int A, B;
char* data = "<serialized-data>";
size_t consumed = Serio::deserialize(data, A, B);
```

If you want to so serialize or deserialize multiple times you can do this:

``` c++
char data[sizeof(int) * 2];
int A = 0;
auto size = Serio::fill(data, A);
int B = 0;
size += Serio::fill(data + size, B);
```

This doesn't slow down the code like in string API case but you should be careful with the size of your buffer.

# Stream API
This API contains 2 functions ```Serio::read()``` and ```Serio::write()``` which are explained below.

You can serialize data types into a output stream of chars. Here is an example of serializing to stream:

``` c++
int A = 0, B = 0;
std::ostringstream stream;
Serio::write(&stream, A, B);
```

You can deserialize data types from a input stream of chars. Here is an example of deserializing from stream:

``` c++
int A, B;
std::istringstream stream;  // contains serialized data
Serio::read(&stream, A, B);
```

If you want to so serialize or deserialize multiple times you can do this:

``` c++
int A = 0;
std::ostringstream stream;
Serio::write(&stream, A);
int B = 0;
Serio::write(&stream, B);
```

This doesn't slow down the code so feel free to use it this way.

In the previous examples we serialize and deserialize ints. Now we'll pay attention to the other data types. We'll use ```serialize()``` and ```deserialize()``` functions from now on but the same logic applies to the other APIs.

# Serio::Size
Size of containers are serialized and deserialized using ```Serio::Size``` type so that the serialized data on one machine can be used on the others. the default size of ```Serio::Size``` is 8 bytes (```uint64_t```) but you can change it to 4 or 2 bytes. Keep in mind that if you change this size and serialize some data you will have to deserialize this data with the size that it was serialized with. This size can be set by setting the macro ```SERIO_SIZE``` and the possible values are 2 or 4 or 8 (default). You shouldn't change this value unless it is nessesary like when you really need the size of output data to be small (the difference in size will depend on the data being serialized but the siza won't drastically reduced so it is preferred to keep the size 8 bytes).

# STL Data Types
Supported STL types are [C++ containers](http://www.cplusplus.com/reference/stl/), std::string, std::complex, std::pair, std::tuple, std::chrono::time_point, std::bitset, std::shared_ptr, std::unique_ptr, std::optional(C++17), std::variant(C++17). Here is an example:

``` c++
std::string A = "Hi";
std::vector<int> B = {1, 2, 3};
std::list<std::vector<int>> C = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}};
std::pair<int, int> D = {1,2};
Serio::ByteArray str = Serio::serialize(A,B,C,D);
```
As you can see you can nest the supported data types and use this library on it. You can deserialize the above code like this:

``` c++
std::string A;
std::vector<int> B;
std::list<std::vector<int>> C;
std::pair<int, int> D;

Serio::ByteArray str = "<serialized-data>";
Serio::deserialize(str, A, B, C, D);
```

# Custom Structures and Classes
You can make Custom classes and structures usable with just one line of code using ```SERIO_REGISTER``` macro. For example:

``` c++
struct Point
{
    int x, y;
    SERIO_REGISTER(x, y)
};

std::list<Point> points = {{0, 0}, {1, 1}};
Serio::ByteArray str = Serio::serialize(points);
```

Here is an example deserialization of above example:

``` c++
std::list<Point> points;
Serio::ByteArray str = "<serialized-data>";
Serio::serialize(points);
```

# Custom Containers
Here is an example of how to make custom containers compatible. Since the size of container is variable we can't use ```SERIO_REGISTER``` macro like we did before. We'll have to implement some of the details ourselves.

``` c++
class Container
{
    std::vector<int> data;

private:
    template <typename Derived>
    friend class Serio::SerializerOps;
    template <typename Derived>
    friend class Serio::DeserializerOps;

    template <typename Serializer>
    inline void _serialize(Serializer& C) const
    {
        C << Serio::Size(data.size());
        for (const auto& value : data) C << value;
    }
    template <typename Deserializer>
    inline void _deserialize(Deserializer& C)
    {
        Serio::Size size;
        C >> size;
        data.resize(size_t(size));
        for (Serio::Size i = 0; i < size; ++i) C >> data[i];
    }
};
```

Here we do the job of ```SERIO_REGISTER``` manually. We define ```_serialize()``` function in your class that serializes size first and then all the items. Then we define ```_deserialize()``` function that deserializes size and resizes the container and then deserializes all the items. Please note that if you serialize size of vector it will have size_t type and it's size will be platform dependent so we use ```Serio::Size``` here so the serialized data will be portable. Now your container is ready to be serialized and deserialized.

# Raw Arrays
You can serialize raw arrays using a structure provided by ```Serio``` which is ```Serio::Array<T>```. Here is an example:

``` c++
int A[10];
Serio::ByteArray str = Serio::serialize(Serio::Array<int>(A, 10));
```

Here is an example deserialization of above example:

``` c++
int A[10];
Serio::ByteArray str = "<serialized-data>";
Serio::deserialize(str, Serio::Array<int>(A, 10));
```

# Advanced Usage
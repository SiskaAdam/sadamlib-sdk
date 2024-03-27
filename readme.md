# SDK For The sadam Library

## About

The `sadam` Library is a collection of externals written for Max. This SDK contains files required in order to write Max externals compatible with The sadam Library.


## Content

- `sadam.stream.h`: Header file to be used by stream-aware objects.


## Writing Stream Aware Externals

By including `sadam.stream.h` in your code, you may author third-party externals that can access or modify data contained by a `sadam.stream`. This short instroduction assumes that you are already familiar with the C language and the Max SDK itself (so that you know how to build a third-party external in C for Max). The `sadam.stream` objects use the globalsymbol mechanism and the notification system presented by the Max SDK Documentation. `sadam.stream` stores the bytes as a C++ `vector` (part of the STL Library), this is the container type which will hold any data queried from the stream and this is the container which you must use if you would like to insert data in your stream. As you will see, it is possible to avoid the usage of a `vector` by sending and/or querying the contents of the stream byte-by-byte, but this is not an efficient, therefore not a recommended way to go.

By including the `sadam.stream.h` header file you will get some common strings used by a stream. If you will not compile your code with a C++ compiler, you will need to call the `sadam_stream_initcommonsymbols` function somewhere in your code (the best choice is in your `ext_main` function) to set these common variables.

To catch any notifications of a stream, you'll have to write a method that responds to the `notify` message (see details in the Max SDK Documentation):

```cpp
void myobject_notify ( t_myobject * x, t_symbol * s, t_symbol * msg,
                       void * sender, void * data ) {
  if ( msg == stream_after_change ) {
    // Do some stuff with the changed stream
  } else if ( msg == stream_before_clear ) {
    // Do some stuff with the stream before clearing it
  }
  // etc...
}
```

A stream will send four types of notifications, two of them can be disabled or enabled by the user (or by your code by invoking the proper method). Apart of this, Max will send notifications when a stream with a particular name was created or destroyed. These are the notifications you can get:

```cpp
t_symbol * stream_binding
t_symbol * stream_unbinding
t_symbol * stream_before_change
t_symbol * stream_after_change
t_symbol * stream_before_clear
t_symbol * stream_after_clear
```

The first two will be sent by Max itself when a stream is bound or unbound to a symbol. This may or may not happen, depending on Max, at each creation/deletion of an instance of `sadam.stream`. The `data` field will contain a pointer to the `t_object` representing the stream that is being bound/unbound.

The next two will be sent when the stream is changed by an `add*`, `erase*`, `insert*` or `replace*` call to the stream and can be enabled or disabled, either by the user or by you, by setting the `notifyonchange` property of the stream. The `data` parameter will contain a pointer to a `vector` containing a copy of the stream before and after it has been changed.

The last two will be sent before and after the stream is being cleared. `stream_before_clear` will pass a pointer to a `vector` containing a copy of the stream in the `data` parameter, however, `stream_after_clear` will pass nothing.

To actually get the notifications, you'll need to register your object with the stream you'd like to listen to. This can be done by invoking the `globalsymbol_reference` method (defined in `ext_globalsymbol.h`):

```cpp
void * stream = globalsymbol_reference ( x, "foo", stream_classname->s_name );
```

In the example above `x` is the pointer to your own object and _foo_ is the name of the stream we wish to listen to. If the stream doesn't exist, we'd get a `NULL` pointer, otherwise we'd get an object pointer to the object holding the _foo_ stream. Of course at some point we'll need to stop listening to the object (at least in our object freeing function). This is achieved by invoking another command:

```cpp
globalsymbol_dereference ( x, "foo", stream_classname->s_name );
```

Remember, for each call of `globalsymbol_reference`, a de-referencing must be called as well.

To read or modify the contents of a named `sadam.stream`, you'll have to invoke one of the internal methods of the class using `object_method` calls. The names of these methods are declared in `sadam.stream.h` and are quite self-explanatory. The parameters required by these calls are documented in the header file itself. For methods that get bytes from the stream (`getbyte` to get a single byte and `getarray` to get an array of bytes) you'll have to provide a pointer to an already initialized variable (either an unsigned char or a vector of unsigned chars). This will hold the return value of your query.

The pointer to the object containing your stream's data, which is required for an `object_method` call, is the one returned by `globalsymbol_reference`. If nonzero, you can invoke a call on one of its methods. Here are some examples:

```cpp
unsigned char myByte;
std::vector<unsigned char> myArray;
void * s = globalsymbol_reference ( x, "foo", stream_classname->s_name );

if ( ! s ) return;
myByte = 0xF2;
myArray.push_back ( 0x3E );
myArray.push_back ( 0x6D );
myArray.push_back ( 0x92 );
object_method ( s, stream_addbyte, myByte);
object_method ( s, stream_addarray, & myArray );
object_method ( s, stream_insertbyte, 2, myByte );
object_method ( s, stream_getbyte, 1, & myByte );
object_method ( s, stream_getarray, 0, 4, & myArray );
object_method ( s, stream_clear );
globalsymbol_dereference ( x, "foo", stream_classname->s_name );
```

After running the above code, `myByte` will contain `0x3E` and `myArray` will be `0xF2`, `0x3E`, `0xF2`, `0x6D`, `0x92`, while the stream itself will be empty.


## Support

This SDK, just like The `sadam` Library, comes free but without any kind of official support or warranty and the author has no responsability for any damage, failure or any other kind of inconvenience that might result from the use of this Library. By using this SDK or The `sadam` Library you automatically agree to the terms above. Please see the included Licence document (in the `etc` folder) for more details.


## Credits

- The sadam Library was done by Ádám Siska.
- The external `sadam.stream` was commissioned by Andrea Szigetvári and the Hungarian Computer Music Foundation.
- I would like to express my most sincere gratitude to my wife, Bea Bartos, and my late Staffordshire Bull Terrier, Tyutyu, for always and constantly cheering me up, even when I did not ask for it.

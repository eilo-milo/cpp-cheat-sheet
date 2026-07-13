C++ Fundamental Data Types & MemoryMemory Basics & sizeofBit: Smallest unit of memory (0 or 1).Byte: Standardized as 8 bits. Memory is addressable byte-by-byte.sizeof operator: Returns the size of an object or type in bytes. The return type is std::size_t.C++#include <iostream>
int main() {
    std::cout << sizeof(int) << '\n'; // Typically prints 4
    return 0;
}
Integer TypesIntegers are integral types that hold whole numbers. By default, integers in C++ are signed (can hold positive, negative, and zero).1. Fundamental IntegersC++ only guarantees minimum sizes for these types. Their actual size depends on the compiler and architecture.short (min 16 bits)int (min 16 bits, typically 32 bits)long (min 32 bits)long long (min 64 bits)2. Fixed-Width Integers (<cstdint>)Best Practice: Use these when you need a guaranteed exact size.std::int16_t / std::uint16_t (Exactly 16 bits)std::int32_t / std::uint32_t (Exactly 32 bits)std::int64_t / std::uint64_t (Exactly 64 bits)⚠️ Warning for 8-bit types: std::int8_t and std::uint8_t are generally treated by the compiler as char (text) rather than integers. Avoid them for math, or use static_cast<int>() when printing them.Integer Best PracticesUse int for general counting or when the exact size doesn't matter.Avoid unsigned integers for math or quantities (even if the value shouldn't be negative). Mixing signed and unsigned math leads to bugs and unexpected wrap-around behavior.Integer Division: Drops the fractional part entirely (e.g., 8 / 5 evaluates to 1).Floating Point TypesFloating-point numbers handle fractions and huge numbers, but they suffer from rounding errors due to precision limits.TypeTypical SizeSignificant Digits (Precision)float4 bytes6 to 9 digitsdouble8 bytes15 to 18 digitslong double8, 12, or 16 bytesVaries heavily (Avoid)Floating Point Best Practices:Always use double unless you have a specific reason (like strict memory limits) to use float.Add a decimal point to floating-point literals to distinguish them from integers: double d { 5.0 };.Use the f suffix for float literals: float f { 5.0f };.Never assume floating-point numbers are exact. They are approximations. Direct equality comparisons (e.g., if (a == 0.1)) are highly prone to rounding errors.Booleans (bool)Holds true (evaluates to 1) or false (evaluates to 0).C++#include <iostream>
int main() {
    bool isReady { true };
    
    // Prints 1 by default
    std::cout << isReady << '\n'; 
    
    // Prints "true" using the boolalpha manipulator
    std::cout << std::boolalpha << isReady << '\n'; 
    return 0;
}
Characters (char)Holds a single ASCII character and is exactly 1 byte.Always use single quotes for characters: 'a'. (Double quotes "a" create a string).Avoid multicharacter literals like '56' or '/n'.Common Escape SequencesSequenceMeaning\nNewline (Use this instead of std::endl)\tHorizontal Tab\'Single Quote\"Double Quote\\BackslashType Conversion & static_castImplicit Conversion: The compiler automatically converts one type to another (e.g., passing an int into a function expecting a double). This can cause warnings if data might be lost (like converting double to int, which drops the decimal).Explicit Conversion (static_cast): Tells the compiler you intentionally want to convert a value, suppressing data-loss warnings.Syntax: static_cast<new_type>(expression)C++#include <iostream>

int main() {
    // 1. Casting double to int (drops the .5)
    int a { static_cast<int>(5.5) }; 
    
    // 2. Casting char to int to see its ASCII numerical value
    char ch { 'A' };
    std::cout << static_cast<int>(ch); // Prints 65
    
    return 0;
}
Miscellaneous Key Typesvoid: Means "no type". Used to indicate a function returns nothing or takes no parameters. You cannot define a variable of type void.std::size_t: Defined in <cstddef>. It is an unsigned integer type returned by the sizeof operator. Used to represent the size or length of objects

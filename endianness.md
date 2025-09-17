- https://www.linkedin.com/posts/t-yashwanth-naidu_embedded-embeddedengineers-embeddedsystems-activity-7373193421072551937-GMLl
  - https://github.com/T-Yashwanth-Naidu/C-Programming-Questions/blob/main/Structures_and_Unions/endianness_conversion/Solution/convert_endian.h
  - https://gcc.gnu.org/onlinedocs/gcc/Byte-Swapping-Builtins.html

```c 
/**
 * @file endian_utils.h
 * @brief Endianness conversion utilities using union-based approach.
 *
 * This module provides functions to convert integers between
 * big-endian and little-endian formats. It supports sizes from
 * 1 to 8 bytes (8, 16, 24, 32, 64-bit values).
 *
 * ## Why Endianness Conversion?
 * - Networking protocols use big-endian (network byte order).
 * - x86 systems are little-endian.
 * - Embedded systems often switch between both (CAN, SPI, I2C).
 *
 * Example:
 *   Input  : 0xAABBCCDD
 *   Output : 0xDDCCBBAA
 * 
 */

#ifndef ENDIAN_UTILS_H
#define ENDIAN_UTILS_H

#include <stdint.h>
#include <stddef.h>

/**
 * @union endian_u
 * @brief Union to access an integer as raw bytes.
 *
 * This union allows interpreting the same memory region as both
 * an integer (`uint64_t`) and an array of bytes (`uint8_t`).
 *
 * Example usage:
 *   endian_u u;
 *   u.value = 0xAABBCCDD;
 *   printf("%X %X %X %X\n", u.bytes[0], u.bytes[1], u.bytes[2], u.bytes[3]);
 */
typedef union {
    uint64_t value;   /**< Integer view (up to 64 bits). */
    uint8_t  bytes[8];/**< Byte-wise view. */
} endian_u;

/**
 * @brief Convert the endianness of an integer.
 *
 * Reverses the byte order of an input integer of given width.
 *
 * @param input       The input integer value (up to 64 bits).
 * @param byte_count  Number of bytes in the integer (1, 2, 3, 4, 8).
 * @return The endian-swapped integer as `uint64_t`.
 *
 * @warning Caller must cast the result to the correct type.
 * @warning Undefined behavior if `byte_count > 8`.
 *
 * ## Edge Cases:
 * - If `byte_count == 1`, the function returns the input unchanged.
 * - Symmetric values (e.g., 0x11223311) remain valid.
 */
static inline uint64_t convert_endian(uint64_t input, size_t byte_count) {
    endian_u src, dst;
    src.value = input;

    for (size_t i = 0; i < byte_count; i++) {
        dst.bytes[i] = src.bytes[byte_count - 1 - i]; // reverse order
    }

    return dst.value;
}

#endif // ENDIAN_UTILS_H
```

```rust
/// Endianness conversion utilities using union-based approach.
///
/// This module provides functions to convert integers between
/// big-endian and little-endian formats. It supports sizes from
/// 1 to 8 bytes (8, 16, 24, 32, 64-bit values).

use std::mem;

/// Union to access an integer as raw bytes.
/// 
/// This union allows interpreting the same memory region as both
/// an integer (u64) and an array of bytes ([u8; 8]).
#[repr(C)]
union EndianU {
    value: u64,
    bytes: [u8; 8],
}

/// Convert the endianness of an integer.
///
/// Reverses the byte order of an input integer of given width.
///
/// # Arguments
/// * `input` - The input integer value (up to 64 bits)
/// * `byte_count` - Number of bytes in the integer (1, 2, 3, 4, 8)
///
/// # Returns
/// The endian-swapped integer as u64
///
/// # Safety
/// This function uses unsafe code to access union fields.
/// The caller must ensure byte_count <= 8.
///
/// # Examples
/// ```
/// let input: u32 = 0xAABBCCDD;
/// let swapped = convert_endian(input as u64, 4);
/// assert_eq!(swapped, 0xDDCCBBAA);
/// ```
pub fn convert_endian(input: u64, byte_count: usize) -> u64 {
    unsafe {
        let mut src = EndianU { value: input };
        let mut dst = EndianU { value: 0 };
        
        for i in 0..byte_count {
            dst.bytes[i] = src.bytes[byte_count - 1 - i];
        }
        
        dst.value
    }
}

/// Alternative safe implementation without unions
pub fn convert_endian_safe(input: u64, byte_count: usize) -> u64 {
    let mut result = 0u64;
    
    for i in 0..byte_count {
        let byte = ((input >> (i * 8)) & 0xFF) as u8;
        result |= (byte as u64) << ((byte_count - 1 - i) * 8);
    }
    
    result
}

/// Convenience functions for common sizes
pub trait EndianConvert {
    fn swap_bytes(self) -> Self;
}

impl EndianConvert for u16 {
    fn swap_bytes(self) -> Self {
        convert_endian(self as u64, 2) as u16
    }
}

impl EndianConvert for u32 {
    fn swap_bytes(self) -> Self {
        convert_endian(self as u64, 4) as u32
    }
}

impl EndianConvert for u64 {
    fn swap_bytes(self) -> Self {
        convert_endian(self, 8)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_8bit_conversion() {
        let input: u8 = 0xAA;
        let result = convert_endian(input as u64, 1);
        assert_eq!(result, 0xAA); // 1 byte remains unchanged
    }

    #[test]
    fn test_16bit_conversion() {
        let input: u16 = 0xAABB;
        let result = convert_endian(input as u64, 2) as u16;
        assert_eq!(result, 0xBBAA);
    }

    #[test]
    fn test_32bit_conversion() {
        let input: u32 = 0xAABBCCDD;
        let result = convert_endian(input as u64, 4) as u32;
        assert_eq!(result, 0xDDCCBBAA);
    }

    #[test]
    fn test_64bit_conversion() {
        let input: u64 = 0x1122334455667788;
        let result = convert_endian(input, 8);
        assert_eq!(result, 0x8877665544332211);
    }

    #[test]
    fn test_safe_implementation() {
        let input: u32 = 0xAABBCCDD;
        let unsafe_result = convert_endian(input as u64, 4) as u32;
        let safe_result = convert_endian_safe(input as u64, 4) as u32;
        assert_eq!(unsafe_result, safe_result);
    }

    #[test]
    fn test_trait_implementation() {
        let val16: u16 = 0xAABB;
        assert_eq!(val16.swap_bytes(), 0xBBAA);
        
        let val32: u32 = 0xAABBCCDD;
        assert_eq!(val32.swap_bytes(), 0xDDCCBBAA);
        
        let val64: u64 = 0x1122334455667788;
        assert_eq!(val64.swap_bytes(), 0x8877665544332211);
    }
}

fn main() {
    // Example usage
    let value32: u32 = 0xAABBCCDD;
    let swapped = convert_endian(value32 as u64, 4) as u32;
    println!("Original: 0x{:08X}", value32);
    println!("Swapped:  0x{:08X}", swapped);
    
    // Using the trait
    let value16: u16 = 0xAABB;
    println!("\nUsing trait:");
    println!("Original u16: 0x{:04X}", value16);
    println!("Swapped u16:  0x{:04X}", value16.swap_bytes());
    
    // Network byte order example
    let host_order: u32 = 0x12345678;
    let network_order = host_order.swap_bytes(); // Assuming little-endian host
    println!("\nNetwork byte order conversion:");
    println!("Host order:    0x{:08X}", host_order);
    println!("Network order: 0x{:08X}", network_order);
}
```

```typescript 
/**
 * Endianness conversion utilities using ArrayBuffer approach.
 *
 * This module provides functions to convert integers between
 * big-endian and little-endian formats. It supports sizes from
 * 1 to 8 bytes (8, 16, 24, 32, 64-bit values).
 * 
 * Since JavaScript doesn't have unions, we use ArrayBuffer and
 * typed arrays to achieve similar byte-level manipulation.
 */

/**
 * Class that mimics the C union behavior using ArrayBuffer
 * Allows accessing the same memory as both integer and bytes
 */
class EndianUnion {
    private buffer: ArrayBuffer;
    private bytesView: Uint8Array;
    private dataView: DataView;

    constructor() {
        // Allocate 8 bytes of memory
        this.buffer = new ArrayBuffer(8);
        this.bytesView = new Uint8Array(this.buffer);
        this.dataView = new DataView(this.buffer);
    }

    /**
     * Set the value as a 64-bit integer (using BigInt)
     */
    set value(val: bigint) {
        this.dataView.setBigUint64(0, val, true); // true = little-endian
    }

    /**
     * Get the value as a 64-bit integer (using BigInt)
     */
    get value(): bigint {
        return this.dataView.getBigUint64(0, true); // true = little-endian
    }

    /**
     * Get the byte array view
     */
    get bytes(): Uint8Array {
        return this.bytesView;
    }

    /**
     * Set value as 32-bit for compatibility
     */
    setValue32(val: number): void {
        this.dataView.setUint32(0, val, true);
    }

    /**
     * Get value as 32-bit for compatibility
     */
    getValue32(): number {
        return this.dataView.getUint32(0, true);
    }
}

/**
 * Convert the endianness of an integer.
 * Reverses the byte order of an input integer of given width.
 * 
 * @param input - The input integer value (number for up to 32 bits, bigint for 64 bits)
 * @param byteCount - Number of bytes in the integer (1, 2, 3, 4, 8)
 * @returns The endian-swapped integer
 */
export function convertEndian(input: number | bigint, byteCount: number): number | bigint {
    const src = new EndianUnion();
    const dst = new EndianUnion();

    // Handle both number and bigint inputs
    if (typeof input === 'bigint') {
        src.value = input;
    } else {
        src.setValue32(input);
    }

    // Reverse the bytes
    for (let i = 0; i < byteCount; i++) {
        dst.bytes[i] = src.bytes[byteCount - 1 - i];
    }

    // Return appropriate type based on byte count
    if (byteCount <= 4) {
        return dst.getValue32() >>> 0; // >>> 0 ensures unsigned
    } else {
        return dst.value;
    }
}

/**
 * Alternative implementation using bitwise operations (for 32-bit values)
 * This is a safe implementation without ArrayBuffer
 */
export function convertEndian32Bit(input: number, byteCount: number): number {
    let result = 0;
    
    for (let i = 0; i < byteCount; i++) {
        const byte = (input >>> (i * 8)) & 0xFF;
        result |= byte << ((byteCount - 1 - i) * 8);
    }
    
    return result >>> 0; // Ensure unsigned
}

/**
 * Convenience functions for common sizes
 */
export class EndianConverter {
    /**
     * Swap bytes of a 16-bit integer
     */
    static swap16(value: number): number {
        return convertEndian(value, 2) as number;
    }

    /**
     * Swap bytes of a 32-bit integer
     */
    static swap32(value: number): number {
        return convertEndian(value, 4) as number;
    }

    /**
     * Swap bytes of a 64-bit integer
     */
    static swap64(value: bigint): bigint {
        return convertEndian(value, 8) as bigint;
    }

    /**
     * Convert from host to network byte order (big-endian)
     * Assumes host is little-endian
     */
    static htonl(value: number): number {
        return this.swap32(value);
    }

    /**
     * Convert from network to host byte order
     * Assumes host is little-endian
     */
    static ntohl(value: number): number {
        return this.swap32(value);
    }

    /**
     * Convert 16-bit host to network byte order
     */
    static htons(value: number): number {
        return this.swap16(value);
    }

    /**
     * Convert 16-bit network to host byte order
     */
    static ntohs(value: number): number {
        return this.swap16(value);
    }
}

/**
 * Helper function to format number as hex string
 */
function toHex(value: number | bigint, padding: number = 8): string {
    const hex = value.toString(16).toUpperCase();
    return '0x' + hex.padStart(padding, '0');
}

// Example usage and tests
if (require.main === module) {
    console.log('=== Endianness Conversion Examples ===\n');

    // 8-bit conversion (no change)
    const byte = 0xAA;
    const byteSwapped = convertEndian(byte, 1);
    console.log(`8-bit:  ${toHex(byte, 2)} -> ${toHex(byteSwapped, 2)}`);

    // 16-bit conversion
    const value16 = 0xAABB;
    const swapped16 = convertEndian(value16, 2);
    console.log(`16-bit: ${toHex(value16, 4)} -> ${toHex(swapped16, 4)}`);

    // 32-bit conversion
    const value32 = 0xAABBCCDD;
    const swapped32 = convertEndian(value32, 4);
    console.log(`32-bit: ${toHex(value32, 8)} -> ${toHex(swapped32, 8)}`);

    // 64-bit conversion
    const value64 = 0x1122334455667788n;
    const swapped64 = convertEndian(value64, 8);
    console.log(`64-bit: ${toHex(value64, 16)} -> ${toHex(swapped64, 16)}`);

    console.log('\n=== Using Convenience Functions ===\n');

    // Using EndianConverter class
    console.log(`swap16: ${toHex(EndianConverter.swap16(0xAABB), 4)}`);
    console.log(`swap32: ${toHex(EndianConverter.swap32(0xAABBCCDD), 8)}`);
    console.log(`swap64: ${toHex(EndianConverter.swap64(0x1122334455667788n), 16)}`);

    console.log('\n=== Network Byte Order Conversion ===\n');
    
    const hostOrder = 0x12345678;
    const networkOrder = EndianConverter.htonl(hostOrder);
    console.log(`Host to Network (htonl): ${toHex(hostOrder, 8)} -> ${toHex(networkOrder, 8)}`);
    console.log(`Network to Host (ntohl): ${toHex(networkOrder, 8)} -> ${toHex(EndianConverter.ntohl(networkOrder), 8)}`);

    console.log('\n=== Testing Alternative Implementation ===\n');
    
    const test32 = 0xAABBCCDD;
    const method1 = convertEndian(test32, 4) as number;
    const method2 = convertEndian32Bit(test32, 4);
    console.log(`ArrayBuffer method: ${toHex(method1, 8)}`);
    console.log(`Bitwise method:     ${toHex(method2, 8)}`);
    console.log(`Results match: ${method1 === method2}`);
}

// Export for testing
export function runTests(): void {
    const tests = [
        { input: 0xAA, bytes: 1, expected: 0xAA },
        { input: 0xAABB, bytes: 2, expected: 0xBBAA },
        { input: 0xAABBCCDD, bytes: 4, expected: 0xDDCCBBAA },
    ];

    console.log('Running tests...\n');
    
    for (const test of tests) {
        const result = convertEndian(test.input, test.bytes) as number;
        const passed = result === test.expected;
        console.log(`Test ${toHex(test.input)}: ${passed ? 'PASSED' : 'FAILED'}`);
        if (!passed) {
            console.log(`  Expected: ${toHex(test.expected)}`);
            console.log(`  Got:      ${toHex(result)}`);
        }
    }

    // Test 64-bit
    const input64 = 0x1122334455667788n;
    const expected64 = 0x8877665544332211n;
    const result64 = convertEndian(input64, 8) as bigint;
    const passed64 = result64 === expected64;
    console.log(`Test 64-bit ${toHex(input64, 16)}: ${passed64 ? 'PASSED' : 'FAILED'}`);
}
```

Shortlist that explains endianness clearly, from quick vids to solid book chapters:

## Watch (quick & clear)

* https://m.youtube.com/watch?v=rXDfuGYVXgc
* https://m.youtube.com/watch?v=nDm1M-QtM-o
* https://www.youtube.com/watch?v=RvFRCDoj6JI
* https://www.youtube.com/watch?v=a9lVoThjV7o
* https://www.youtube.com/watch?v=lWi8bFV0N0Q
* https://www.youtube.com/watch?v=nvKKogL-aVc
* https://www.youtube.com/watch?v=Lff8x-P7NJU, https://github.com/omar-samin/Computer-Organization-Assembly-Language
* **Computerphile – “Endianness Explained With an Egg.”** Visual, memorable intro to big vs little endian. ([YouTube][1])
* **Jacob Sorber – “Dealing with Endianness Issues in your Programs.”** Practical: htons/ntohl, when conversions matter. ([YouTube][2])
* **Ben Eater – “the TRUTH about numbers and network programming.”** Great intuition for byte order in networking contexts. ([YouTube][3])
* **Byte Order (Endianness).** Straightforward classroom-style explainer if you want a second pass. ([YouTube][4])

## Read (short, hands-on articles)

* https://www.linkedin.com/pulse/counting-from-zero-endianness-file-systems-forensic-cost-jeff-hamm-oxeae
* **Endian Issues** (from *Programming Embedded Systems*, O’Reilly) – concise embedded-focused section. ([O'Reilly Media][5])
* **Endianness & Serial Communication** (EmbeddedRelated) – framing bytes across links (UART/SPI/etc.). ([embeddedrelated.com][6])
* **Embetronicx guide** – microcontroller examples with diagrams. ([embetronicx.com][7])

## Book chapters (deeper understanding)

* **CS\:APP (Bryant & O’Hallaron)** – “Addressing and Byte Ordering” section; gold standard for how memory & bytes are laid out. ([shuyuej.com][8])
* **TCP/IP Illustrated, Vol. 1 (Stevens)** – explains why “network byte order” is big-endian and shows real packet fields. ([R-5][9])
* **Computer Organization & Design (Patterson & Hennessy)** – architectural perspective on byte ordering. ([Archive.org][10])

[1]: https://www.youtube.com/watch?v=NcaiHcBvDR4 "Endianness Explained With an Egg - Computerphile"
[2]: https://www.youtube.com/watch?v=OoHich9BPxg "Dealing with Endianness Issues in your Programs"
[3]: https://www.youtube.com/watch?v=_yRW4nQFrB8 "the TRUTH about numbers and network programming ..."
[4]: https://www.youtube.com/watch?v=CounrFEsOeA "Byte Order (Endianness)"
[5]: https://www.oreilly.com/library/view/programming-embedded-systems/0596009836/ch06s03.html "Endian Issues - Programming Embedded Systems, 2nd ..."
[6]: https://www.embeddedrelated.com/showarticle/174.php "Endianness and Serial Communication - Stephen Friederichs"
[7]: https://embetronicx.com/tutorials/p_language/c/little-endian-and-big-endian/ "Little Endian and Big Endian | Full Guide with Examples ..."
[8]: https://shuyuej.com/books/Computer%20Systems-%20A%20Programmer%27s%20Perspective.pdf "Computer Systems: A Programmer's Perspective"
[9]: https://www.r-5.org/files/books/computers/internals/net/Richard_Stevens-TCP-IP_Illustrated-EN.pdf "TCP/IP Illustrated, Volume 1: The Protocols"
[10]: https://ia601209.us.archive.org/24/items/ComputerOrganizationAndDesign3rdEdition/-computer%20organization%20and%20design%203rd%20edition.pdf "Computer Organization and Design"

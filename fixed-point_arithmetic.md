```c 
#define SCALE_FACTOR 10
unsigned char A = 5; //this is really 0.5
unsigned char B = 6; //this is really 0.6
unsigned char C;
void main()
{
    C = A + B;
    //C is now 11 with integer math
    //but is really 1.1 when we
    //take into account our
    //scale factor!
}
```

```rust 
// Define the scale factor as a constant
const SCALE_FACTOR: u8 = 10;

fn main() {
    // Using u8 (unsigned 8-bit integer) equivalent to unsigned char
    let a: u8 = 5;  // this is really 0.5
    let b: u8 = 6;  // this is really 0.6
    
    let c: u8 = a + b;
    
    // C is now 11 with integer math
    // but is really 1.1 when we take into account our scale factor!
    
    println!("Raw values:");
    println!("A = {}", a);
    println!("B = {}", b);
    println!("C = {}", c);
    
    println!("\nReal values (after scaling):");
    println!("A = {}", a as f32 / SCALE_FACTOR as f32);
    println!("B = {}", b as f32 / SCALE_FACTOR as f32);
    println!("C = {}", c as f32 / SCALE_FACTOR as f32);
}

// Alternative: A more idiomatic Rust approach using a struct
#[derive(Debug, Clone, Copy)]
struct FixedPoint {
    value: u8,
}

impl FixedPoint {
    const SCALE: u8 = 10;
    
    fn new(value: u8) -> Self {
        FixedPoint { value }
    }
    
    fn from_float(f: f32) -> Self {
        FixedPoint {
            value: (f * Self::SCALE as f32) as u8,
        }
    }
    
    fn to_float(&self) -> f32 {
        self.value as f32 / Self::SCALE as f32
    }
    
    fn add(&self, other: &FixedPoint) -> FixedPoint {
        FixedPoint {
            value: self.value + other.value,
        }
    }
}

// Example using the struct approach
fn fixed_point_example() {
    let a = FixedPoint::new(5);  // 0.5
    let b = FixedPoint::new(6);  // 0.6
    let c = a.add(&b);            // 1.1
    
    println!("\nUsing FixedPoint struct:");
    println!("A = {} (real: {})", a.value, a.to_float());
    println!("B = {} (real: {})", b.value, b.to_float());
    println!("C = {} (real: {})", c.value, c.to_float());
}
```
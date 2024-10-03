# RuMono Bugs
This is the collection of bugs detected by RuMono. RuMono is a tool to synthesize fuzz drivers (harnesses) for the Rust library, with the support for Rust generic system. We utilize RuMono to test 29 Rust libraries and detect 23 bugs, 18 of which are related to generic APIs. We have reported all bugs to maintainers by creating bug issues in their github repositories. 

Here is the overview of detected bugs. We list all bug details below, including links to issues, affected versions, involved generic APIs and other information, grouped by library. Click the link to navigate to the corresponding bug report.

| Bug ID            | Library      | Category      | Issue URL                                                | note           |
| ----------------- | ------------ | ------------- | -------------------------------------------------------- | -------------- |
| [bug-01](#bug-01) | regex        | utf-8         | https://github.com/rust-lang/regex/issues/1006           |                |
| [bug-02](#bug-02) | regex        | utf-8         | https://github.com/rust-lang/regex/issues/1006           |                |
| [bug-03](#bug-03) | form-encoded | utf-8         | https://github.com/servo/rust-url/issues/872             |                |
| [bug-04](#bug-04) | monero-rs    | OOB           | https://github.com/monero-rs/monero-rs/issues/193        |                |
| [bug-05](#bug-05) | monero-rs    | OOM           | https://github.com/monero-rs/monero-rs/issues/193        |                |
| [bug-06](#bug-06) | monero-rs    | overflow      | https://github.com/monero-rs/monero-rs/issues/193        |                |
| [bug-07](#bug-07) | lofty        | unreachable   | https://github.com/Serial-ATA/lofty-rs/issues/295        |                |
| [bug-08](#bug-08) | lofty        | overflow      | https://github.com/Serial-ATA/lofty-rs/issues/295        |                |
| [bug-09](#bug-09) | odoh_rs      | OOM           | https://github.com/cloudflare/odoh-rs/issues/30          | false positive |
| [bug-10](#bug-10) | odoh_rs      | OOM           | https://github.com/cloudflare/odoh-rs/issues/30          | false positive |
| [bug-11](#bug-11) | neli         | overflow      | https://github.com/jbaublitz/neli/issues/236             |                |
| [bug-12](#bug-12) | neli         | overflow      | https://github.com/jbaublitz/neli/issues/236             |                |
| [bug-13](#bug-13) | neli         | overflow      | https://github.com/jbaublitz/neli/issues/236             |                |
| [bug-14](#bug-14) | neli         | OOM           | https://github.com/jbaublitz/neli/issues/236             |                |
| [bug-15](#bug-15) | html2text    | OOM           | https://github.com/jugglerchris/rust-html2text/issues/93 | false positive |
| [bug-16](#bug-16) | html2text    | overflow      | https://github.com/jugglerchris/rust-html2text/issues/93 |                |
| [bug-17](#bug-17) | html2text    | unwrap        | https://github.com/jugglerchris/rust-html2text/issues/93 | misuse         |
| [bug-18](#bug-18) | html2text    | unreachable   | https://github.com/jugglerchris/rust-html2text/issues/93 | misuse         |
| [bug-19](#bug-19) | html2text    | infinite loop | https://github.com/jugglerchris/rust-html2text/issues/93 |                |
| [bug-20](#bug-20) | xorfilter    | overflow      | https://github.com/prataprc/xorfilter/issues/33          |                |
| [bug-21](#bug-21) | xorfilter    | OOB           | https://github.com/prataprc/xorfilter/issues/33          |                |
| [bug-22](#bug-22) | num          | overflow      | https://github.com/rust-num/num/issues/431               | false positive |
| [bug-23](#bug-23) | tui          | overflow      | n/a                                                      |                |


## regex

Issue URL: https://github.com/rust-lang/regex/issues/1006

Version: `1.8.3`

Category: `utf-8`

Involved generic API: `regex::Regex::replace`, `regex::Regex::replace_all`

Description:  

- Bug-01, bug-02 panicked at "byte index x is not a char boundary" in the generic API `regex::Regex::replace` and `regex::Regex::replace_all`.

### PoC

#### bug-01

```rust
fn main() {
    let re = regex::Regex::new(r"\B|00(?-u)\B").unwrap();
    let text = r"𐾁00000000";
    let rep = r"0𐾁Ű000ο";
    let _ = re.replace(text, rep);
}
```

#### bug-02

```rust
fn main(){
    let re = regex::Regex::new(r"\B|(?-u)\B0").unwrap();
    let _ = re.replace_all("00000^睓00" ,"00000000000");
}
```

Response: The developer confirmed and fixed these bugs in `regex 1.8.4`.



## form_urlencoded

Issue URL: https://github.com/servo/rust-url/issues/872

Version: `1.2.0`

Category: `utf-8`

Involved generic API: `form_urlencoded::Serializer::for_suffix`

Bug description: 

- Bug-03 panicked at "assertion failed: self.is_char_boundary(new_len)". The generic API `for_suffix`  will cause unexpected panic when receiving certain utf-8 string.

### PoC

#### bug-03

```rust
use std::string::String;
use form_urlencoded::Serializer;
fn foo(){
    let s=String::from("é0");
    let mut serializer= Serializer::for_suffix(s, 1);
    let _ = serializer.clear();
}
```

Response: The developer has not responded to these bugs.



## monero-rs

Issue URL: https://github.com/monero-rs/monero-rs/issues/193

Version: `0.19.0`

Category: `OOB`,`overflow`,`OOM`

Involved generic API: `monero::consensus::encode::Decodable::consensus_decode`, 

Bug description: 

- bug-04 panicked at "index out of bounds" in generic API `consensus_decode`. Several monomorphic APIs of `consensus_decode` can trigger this panic. We show one of these cases in our report. 
- bug-05 causes an memory allocation fail.
- bug-06 panicked at "attempt to subtract with overflow". Like bug-04, several monomorphic APIs of `consensus_decode` can trigger this panic.

### PoC

#### bug-04

```rust
fn main(){
    let data=[128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 128, 101, 50, 55, 52, 97, 57, 99, 0, 255, 49, 100, 54, 50, 49, 49, 99, 102, 54, 128, 128, 128, 128, 128, 128, 128, 101, 50, 55, 52, 97, 57, 99, 0, 255, 49, 100, 54, 50, 49, 49, 99, 102, 54, 56, 49, 98, 53, 128];
    let _local0_param0_helper1 = &mut (&data[..]);
    let _local0: std::result::Result::<monero::util::address::Address, monero::consensus::encode::Error> = <monero::util::address::Address as monero::consensus::encode::Decodable>::consensus_decode(_local0_param0_helper1);
}
```

#### bug-05

```rust
fn main(){
    let data=[101, 55, 102, 55, 102, 55, 52, 101, 100, 53, 54, 56, 51, 49, 54, 99, 98, 102, 48, 48, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 42, 0, 1, 255, 140, 216, 216, 216, 216, 216, 216, 216, 156, 156, 156, 156, 156, 156, 157, 156, 156, 156, 156, 161, 156, 156, 156, 156, 156, 255, 1, 0, 0, 3, 98, 52, 99, 51, 101, 50, 57, 100, 101, 0, 64, 57, 255, 255, 255, 127, 100, 100, 49, 49, 41, 255, 0, 0, 0, 0, 0, 0, 255, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 49, 2, 2, 2, 2, 2, 0, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 101, 51, 50, 97, 48, 48, 2, 2, 2, 2, 100, 49, 2, 2, 2];
    let _local0_param0_helper1 = &mut (&data[..]);
    let _local0: std::result::Result::<monero::blockdata::block::Block, monero::consensus::encode::Error> = <monero::blockdata::block::Block as monero::consensus::encode::Decodable>::consensus_decode(_local0_param0_helper1);
}
```

#### bug-06

```rust
fn main(){
    let data=[98, 52, 99, 51, 101, 50, 57, 100, 101, 255, 64, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 49, 2, 2, 2, 2, 0, 0, 0, 0, 0, 255, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 49, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 236, 1, 2, 3, 2, 243, 255, 0, 0, 0, 25, 0, 4, 0, 53, 97, 55, 98, 128, 0, 2, 2, 2, 2, 2, 0, 255, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 49, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 3, 232, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 21, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 73, 36, 146, 73, 36, 146, 74, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 250, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 128, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 6, 255, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 50, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 0, 0, 64, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 100, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 0, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 98, 52, 99, 51, 101, 50, 57, 100, 101, 0, 64, 49, 2, 2, 2, 2, 0, 0, 0, 0, 0, 255, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 10, 0, 0, 0, 0, 0];
    let _local0_param0_helper1 = &mut (&data[..]);
    let _local0: std::result::Result::<monero::blockdata::block::Block, monero::consensus::encode::Error> = <monero::blockdata::block::Block as monero::consensus::encode::Decodable>::consensus_decode(_local0_param0_helper1);
}
```

Response: The developer confirmed and fixed these bugs in `monero 0.20.0`.



## lofty

Issue URL: https://github.com/Serial-ATA/lofty-rs/issues/295

Version: `0.17.1`

Category: `overflow`,`unreachable`

Involved generic API: `lofty::id3::v2::ExtendedTextFrame::parse`, `lofty::id3::v2::RelativeVolumeAdjustmentFrame::parse`

Bug description: 

- bug-07 panicked at "internal error: entered unreachable code".
- bug-08 panicked at "attempt to add with overflow".

### PoC

#### bug-07

```rust
let data=[1,0,0,0];
let _local0 = lofty::id3::v2::Id3v2Tag::new();
let _local1_param0_helper1 = &(_local0);
let _local1 = lofty::id3::v2::Id3v2Tag::original_version(_local1_param0_helper1);
let _local2_param0_helper1 = &mut (&data[..]);
let _: lofty::error::Result::<std::option::Option::<lofty::id3::v2::ExtendedTextFrame>> = lofty::id3::v2::ExtendedTextFrame::parse(_local2_param0_helper1, _local1);
```

#### bug-08

```rust
let data=[57, 25, 25, 0, 4, 1, 54, 0, 51, 6, 6, 6, 25, 25, 25, 129, 6, 151, 28, 25, 25, 0, 51, 51, 50, 5, 5, 5, 26, 5, 5, 25, 6, 6, 25, 26, 246, 25, 25, 129, 6, 151, 3, 252, 56, 0, 53, 56, 55, 52];
let _local0 = <lofty::ParsingMode as std::default::Default>::default();
let _local1_param0_helper1 = &mut (&data[..]);
let _: lofty::error::Result::<std::option::Option::<lofty::id3::v2::RelativeVolumeAdjustmentFrame>> = lofty::id3::v2::RelativeVolumeAdjustmentFrame::parse(_local1_param0_helper1, _local0);
```

Response: The developer confirmed and fixed these bugs.



## odoh_rs

Issue URL: https://github.com/cloudflare/odoh-rs/issues/30

Version: `0.17.1`

Category: `OOM`

Involved generic API: ` odoh_rs::ObliviousDoHMessagePlaintext::new`

Bug description: 

- bug-09 panicked at `capacity overflow`.
- bug-10 causes a memory allocation failure.

### PoC

#### bug-09

```rust
let _local0 = <std::vec::Vec::<u8, std::alloc::Global> as std::convert::From::<&str>>::from("0");
let _local1: odoh_rs::ObliviousDoHMessagePlaintext = odoh_rs::ObliviousDoHMessagePlaintext::new(_local0, 17625665026532540416);
let _ = odoh_rs::ObliviousDoHMessagePlaintext::into_msg(_local1);
```

#### bug-10

```rust
let _local0 = <std::vec::Vec::<u8, std::alloc::Global> as std::convert::From::<&str>>::from("0");
let _local1: odoh_rs::ObliviousDoHMessagePlaintext = odoh_rs::ObliviousDoHMessagePlaintext::new(_local0, 3617290116880544306);
let _ = odoh_rs::ObliviousDoHMessagePlaintext::into_msg(_local1);
```

Response: The developer considered bug-07 and bug-08 as false positive, as the given input of API is too big. The developer preferred it is caller's responsibility to decide the proper padding size.

## neli

Issue URL: https://github.com/jbaublitz/neli/issues/236

Version: `0.7.0-rc2`

Category: `overflow`,`OOM`

Involved generic API: none

Bug description: 

- bug-11, bug-12 and bug-13 cause distinct overflow crashes with different certain input.
- bug-14 causes a memory allocation failure.

### PoC

#### bug-11

```rust
let data = [0, 0, 0, 0];
let _local0 = neli::utils::Groups::new_groups(&data[..]);
let _local1_param0_helper1 = &(_local0);
let _local1 = neli::utils::Groups::as_groups(_local1_param0_helper1);
let _local2_param0_helper1 = &(_local1);
let _: usize = <std::vec::Vec<u32> as neli::Size>::unpadded_size(_local2_param0_helper1);
```

#### bug-12

```rust
let data = [78, 122, 122, 122, 122, 250, 104, 122, 122, 122, 122, 56];
let _local0 = neli::utils::Groups::new_groups(&data[..]);
let _local1_param0_helper1 = &(_local0);
let _local1 = neli::utils::Groups::as_groups(_local1_param0_helper1);
let _local2_param0_helper1 = &(_local1);
let _: usize = <std::vec::Vec<u32> as neli::Size>::unpadded_size(_local2_param0_helper1);
```

#### bug-13

```rust
let _local0 = neli::utils::NetlinkBitArray::new(0xffffffffffffffff);
let _local1_param0_helper1 = &(_local0);
let _local1 = neli::utils::NetlinkBitArray::to_vec(_local1_param0_helper1);
let _local2_param0_helper1 = &(_local1);
let _: usize = <std::vec::Vec::<u32> as neli::Size>::unpadded_size(_local2_param0_helper1);
```

#### bug-14

```rust
let _local0 = neli::utils::NetlinkBitArray::new(361700864146343200);
let _local1_param0_helper1 = &(_local0);
let _local1 = neli::utils::NetlinkBitArray::to_vec(_local1_param0_helper1);
let _local2_param0_helper1 = &(_local1);
let _: usize = <std::vec::Vec::<u32> as neli::Size>::unpadded_size(_local2_param0_helper1);
```

Response: The developer has not responded to these bugs.



## html2text

Issue URL: https://github.com/jugglerchris/rust-html2text/issues/93

Version: `0.6.0`

Catogery: `OOM`,`overflow`,`unwrap`,`infinite loop`,`unreachable`

Involved generic API: `html2text::render::text_renderer::SubRenderer::new`, `html2text::render::text_renderer::SubRenderer::new_sub_renderer`, `html2text::render::text_renderer::SubRenderer::add_horizontal_border`, `html2text::parse`, `html2text::RenderTree::render`, `html2text::render::text_renderer::SubRenderer::end_strikeout`, `html2text::render::text_renderer::SubRenderer::end_pre`

Bug description: 

- Bug-15 panicked at "capacity overflow".
- Bug-16 panicked at "attempt to subtract with overflow".
- Bug-17 panicked at "called `Option::unwrap()` on a `None` value"
- Bug-18 panicked at "Attempt to end a preformatted block which wasn't opened".
- Bug-19 causes an infinite loop.

### PoC

#### bug-15

```rust
let _local0 = html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::RichDecorator>::new(11787190863583748771, html2text::render::text_renderer::RichDecorator{});
let mut _local1 = <html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::RichDecorator> as html2text::render::Renderer>::new_sub_renderer(&_local0, 11791448176899352125);
let _ = <html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::RichDecorator> as html2text::render::Renderer>::add_horizontal_border(&mut _local1);
```

#### bug-16

```rust
let data=[60, 116, 97, 98, 108, 101, 62, 60, 116, 114, 62, 60, 116, 100, 62, 120, 105, 60, 48, 62, 0, 0, 0, 60, 116, 97, 98, 108, 101, 62, 58, 58, 58, 62, 58, 62, 62, 62, 58, 60, 112, 32, 32, 32, 32, 32, 32, 32, 71, 87, 85, 78, 16, 16, 62, 60, 15, 16, 16, 16, 16, 16, 16, 15, 38, 16, 16, 16, 15, 1, 16, 16, 16, 16, 16, 16, 162, 111, 107, 99, 91, 112, 57, 64, 94, 100, 60, 111, 108, 47, 62, 127, 60, 108, 73, 62, 125, 109, 121, 102, 99, 122, 110, 102, 114, 98, 60, 97, 32, 104, 114, 101, 102, 61, 98, 111, 103, 32, 105, 100, 61, 100, 62, 60, 111, 15, 15, 15, 15, 15, 15, 15, 39, 15, 15, 15, 106, 102, 59, 99, 32, 32, 32, 86, 102, 122, 110, 104, 93, 108, 71, 114, 117, 110, 100, 96, 121, 57, 60, 107, 116, 109, 247, 62, 60, 32, 60, 122, 98, 99, 98, 97, 32, 119, 127, 127, 62, 60, 112, 62, 121, 116, 60, 47, 116, 100, 62, 62, 60, 111, 98, 62, 123, 110, 109, 97, 101, 105, 119, 60, 112, 101, 101, 122, 102, 63, 120, 97, 62, 60, 101, 62, 60, 120, 109, 112, 32, 28, 52, 55, 50, 50, 49, 52, 185, 150, 99, 62, 255, 112, 76, 85, 60, 112, 62, 73, 100, 116, 116, 60, 75, 50, 73, 116, 120, 110, 127, 255, 118, 32, 42, 40, 49, 33, 112, 32, 36, 107, 57, 60, 5, 163, 62, 49, 55, 32, 33, 118, 99, 63, 60, 109, 107, 43, 119, 100, 62, 60, 104, 58, 101, 163, 163, 163, 163, 220, 220, 220, 220, 220, 220, 220, 220, 220, 220, 220, 220, 1, 107, 117, 107, 108, 44, 102, 58, 60, 116, 101, 97, 106, 98, 59, 60, 115, 109, 52, 58, 115, 98, 62, 232, 110, 114, 32, 60, 117, 93, 120, 112, 119, 111, 59, 98, 120, 61, 206, 19, 61, 206, 19, 59, 1, 110, 102, 60, 115, 0, 242, 64, 203, 8, 111, 50, 59, 121, 122, 32, 42, 35, 32, 37, 101, 120, 104, 121, 0, 242, 59, 63, 121, 231, 130, 130, 130, 170, 170, 1, 32, 0, 0, 0, 28, 134, 200, 90, 119, 48, 60, 111, 108, 118, 119, 116, 113, 59, 100, 60, 117, 43, 110, 99, 9, 216, 157, 137, 216, 157, 246, 167, 62, 60, 104, 61, 43, 28, 134, 200, 105, 119, 48, 60, 122, 110, 0, 242, 61, 61, 114, 231, 130, 130, 130, 170, 170, 170, 233, 222, 222, 162, 163, 163, 163, 163, 163, 163, 163, 85, 100, 116, 99, 61, 60, 163, 163, 163, 163, 163, 220, 220, 1, 109, 112, 105, 10, 59, 105, 220, 215, 10, 59, 122, 100, 100, 121, 97, 43, 43, 43, 102, 122, 100, 60, 62, 114, 116, 122, 115, 61, 60, 115, 101, 62, 215, 215, 215, 215, 215, 98, 59, 60, 109, 120, 57, 60, 97, 102, 113, 229, 43, 43, 43, 43, 43, 43, 43, 43, 43, 35, 43, 43, 101, 58, 60, 116, 98, 101, 107, 98, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 98, 99, 62, 60, 112, 102, 59, 124, 107, 111, 97, 98, 108, 118, 60, 116, 102, 101, 104, 97, 62, 60, 255, 127, 46, 60, 116, 101, 62, 60, 105, 102, 63, 116, 116, 60, 47, 116, 101, 62, 62, 60, 115, 98, 62, 123, 109, 108, 97, 100, 119, 118, 60, 111, 99, 97, 103, 99, 62, 60, 255, 127, 46, 60, 103, 99, 62, 60, 116, 98, 63, 60, 101, 62, 60, 109, 109, 231, 130, 130, 130, 213, 213, 213, 233, 222, 222, 59, 101, 103, 58, 60, 100, 111, 61, 65, 114, 104, 60, 47, 101, 109, 62, 60, 99, 99, 172, 97, 97, 58, 60, 119, 99, 64, 126, 118, 104, 100, 100, 107, 105, 60, 120, 98, 255, 255, 255, 0, 60, 255, 127, 46, 60, 113, 127];
let _local0: html2text::RenderTree = html2text::parse(&data[..]);
let _local1: html2text::RenderedText::<html2text::render::text_renderer::RichDecorator> = html2text::RenderTree::render(_local0, 1, html2text::render::text_renderer::RichDecorator{});
```

#### bug-17

```rust
let mut _local0: html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::TrivialDecorator> = html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::TrivialDecorator>::new(18446744073709551615, html2text::render::text_renderer::TrivialDecorator{});
let _local1_param0_helper1 = &mut (_local0);
let _ = <html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::TrivialDecorator> as html2text::render::Renderer>::end_strikeout(_local1_param0_helper1);
```

#### bug-18

```rust
let _local0: html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::RichDecorator> = html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::RichDecorator>::new(14829735431805717965, html2text::render::text_renderer::RichDecorator{});
let _local1_param0_helper1 = &(_local0);
let mut _local1: html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::RichDecorator> = <html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::RichDecorator> as html2text::render::Renderer>::new_sub_renderer(_local1_param0_helper1, 14829735431805717810);
let _local2_param0_helper1 = &mut (_local1);
let _ = <html2text::render::text_renderer::SubRenderer::<html2text::render::text_renderer::RichDecorator> as html2text::render::Renderer>::end_pre(_local2_param0_helper1);
```

#### bug-19

```rust
let data=[60, 116, 97, 98, 108, 101, 62, 60, 116, 114, 62, 60, 116, 100, 62, 120, 105, 60, 48, 62, 0, 0, 0, 60, 116, 97, 98, 108, 101, 62, 58, 58, 58, 62, 58, 62, 62, 62, 58, 60, 112, 32, 32, 32, 32, 32, 32, 32, 71, 87, 85, 78, 16, 16, 62, 60, 15, 16, 16, 16, 16, 16, 16, 15, 38, 16, 16, 16, 15, 1, 16, 16, 16, 16, 16, 16, 162, 111, 107, 99, 91, 112, 57, 64, 94, 100, 60, 111, 108, 47, 62, 127, 60, 108, 73, 62, 125, 109, 121, 102, 99, 122, 110, 102, 114, 98, 60, 97, 32, 104, 114, 101, 102, 61, 98, 111, 103, 32, 105, 100, 61, 100, 62, 60, 111, 15, 15, 15, 15, 15, 15, 15, 39, 15, 15, 15, 106, 102, 59, 99, 32, 32, 32, 86, 102, 122, 110, 104, 93, 108, 71, 114, 117, 110, 100, 96, 121, 57, 60, 107, 116, 109, 247, 62, 60, 32, 60, 122, 98, 99, 98, 97, 32, 119, 127, 127, 62, 60, 112, 62, 121, 116, 60, 47, 116, 100, 62, 62, 60, 111, 98, 62, 123, 110, 109, 97, 101, 105, 119, 60, 112, 101, 101, 122, 102, 63, 120, 97, 62, 60, 101, 62, 60, 120, 109, 112, 32, 28, 52, 55, 50, 50, 49, 52, 185, 150, 99, 62, 255, 112, 76, 85, 60, 112, 62, 73, 100, 116, 116, 60, 75, 50, 73, 116, 120, 110, 127, 255, 118, 32, 42, 40, 49, 33, 112, 32, 36, 107, 57, 60, 5, 163, 62, 49, 55, 32, 33, 118, 99, 63, 60, 109, 107, 43, 119, 100, 62, 60, 104, 58, 101, 163, 163, 163, 163, 220, 220, 220, 220, 220, 220, 220, 220, 220, 220, 220, 220, 1, 107, 117, 107, 108, 44, 102, 58, 60, 116, 101, 97, 106, 98, 59, 60, 115, 109, 52, 58, 115, 98, 62, 232, 110, 114, 32, 60, 117, 93, 120, 112, 119, 111, 59, 98, 120, 61, 206, 19, 61, 206, 19, 59, 1, 110, 102, 60, 115, 0, 242, 64, 203, 8, 111, 50, 59, 121, 122, 32, 42, 35, 32, 37, 101, 120, 104, 121, 0, 242, 59, 63, 121, 231, 130, 130, 130, 170, 170, 1, 32, 0, 0, 0, 28, 134, 200, 90, 119, 48, 60, 111, 108, 118, 119, 116, 113, 59, 100, 60, 117, 43, 110, 99, 9, 216, 157, 137, 216, 157, 246, 167, 62, 60, 104, 61, 43, 28, 134, 200, 105, 119, 48, 60, 122, 110, 0, 242, 61, 61, 114, 231, 130, 130, 130, 170, 170, 170, 233, 222, 222, 162, 163, 163, 163, 163, 163, 163, 163, 85, 100, 116, 99, 61, 60, 163, 163, 163, 163, 163, 220, 220, 1, 109, 112, 105, 10, 59, 105, 220, 215, 10, 59, 122, 100, 100, 121, 97, 43, 43, 43, 102, 122, 100, 60, 62, 114, 116, 122, 115, 61, 60, 115, 101, 62, 215, 215, 215, 215, 215, 98, 59, 60, 109, 120, 57, 60, 97, 102, 113, 229, 43, 43, 43, 43, 43, 43, 43, 43, 43, 35, 43, 43, 101, 58, 60, 116, 98, 101, 107, 98, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 43, 98, 99, 62, 60, 112, 102, 59, 124, 107, 111, 97, 98, 108, 118, 60, 116, 102, 101, 104, 97, 62, 60, 255, 127, 46, 60, 116, 101, 62, 60, 105, 102, 63, 116, 116, 60, 47, 116, 101, 62, 62, 60, 115, 98, 62, 123, 109, 108, 97, 100, 119, 118, 60, 111, 99, 97, 103, 99, 62, 60, 255, 127, 46, 60, 103, 99, 62, 60, 116, 98, 63, 60, 101, 62, 60, 109, 109, 231, 130, 130, 130, 213, 213, 213, 233, 222, 222, 59, 101, 103, 58, 60, 100, 111, 61, 65, 114, 104, 60, 47, 101, 109, 62, 60, 99, 99, 172, 97, 97, 58, 60, 119, 99, 64, 126, 118, 104, 100, 100, 107, 105, 60, 120, 98, 255, 255, 255, 0, 60, 255, 127, 46, 60, 113, 127];
let _local0 = html2text::parse(&data[..]);
let d1 = TrivialDecorator::new();
let _local1 = html2text::RenderTree::render(_local0, 1, d1);
```

Response: The developer considered bug-15 as the false positive since the given input of `html2text::render::text_renderer::SubRenderer::new` is requiring an unreasonable amount of RAM in this bug. The developer considered bug-17 and bug-18 as the misuse of low-level APIs, yet it reveals that the library exposes some low-level APIs to the public, so that allows users to misuse these APIs. We suggest adding some assertions to provide clear information for users, or consider configuring these low-level APIs to private. The rest of the bugs are confirmed and fixed by the maintainer. 

## xorfilter

Issue URL: https://github.com/prataprc/xorfilter/issues/33

Version: `0.5.1`

Category: `overflow`, `OOB`

Involved generic API: `xorfilter::Fuse8::with_hasher`, `xorfilter::Xor8::with_hasher`, `xorfilter::Xor8::contains_key`.

Bug description: 

- bug-20 panicked at "attempt to add with overflow" panic.
- bug-21 panicked at "index out of bounds" panic.

### PoC

#### bug-20

```rust
let mut _local0: xorfilter::Fuse8::<xorfilter::NoHash> = xorfilter::Fuse8::<xorfilter::NoHash>::with_hasher(4177066232, xorfilter::NoHash{});
```

#### bug-21

```rust
let _local0: xorfilter::Xor8::<xorfilter::NoHash> = xorfilter::Xor8::<xorfilter::NoHash>::with_hasher(xorfilter::NoHash{});
let _local1_param0_helper1 = &(_local0);
let _: bool = xorfilter::Xor8::<xorfilter::NoHash>::contains_key(_local1_param0_helper1, 14355285626356002360);
```

Response: The developer has not responded to these bugs so far.



## num

Issue URL: https://github.com/rust-num/num/issues/431

Version: `0.5.1`

Category: `overflow`,

Involved generic API: `num::integer::mod_floor`

Bug description: 

- Bug-22 panic at `attempt to calculate the remainder with overflow`.

### PoC

#### bug-22

```rust
let _: i8 = num::integer::mod_floor(-128, -1); 
```

Response:  The developer considered bug-22 a false positive since the panic is consistent behavior with Rust's `%` operator, e.g., `i8::MIN % -1` also panics.



## tui

Issue URL: We cannot issue our bug since this crate  has been public archived.

Version: `0.19.0`

Category: `overflow`

Involved generic API: 

Bug description: 

- Bug-23 panicked at "attempt to add with overflow".

### PoC

#### bug-23

```rust
let _local0 = <std::vec::Vec::<u8> as std::convert::From::<&str>>::from("0");
let mut _local1: tui::backend::CrosstermBackend::<std::vec::Vec::<u8>> = tui::backend::CrosstermBackend::<std::vec::Vec::<u8>>::new(_local0);
let _local2_param0_helper1 = &mut (_local1);
let _: std::io::Result::<()> = <tui::backend::CrosstermBackend::<std::vec::Vec::<u8>> as tui::backend::Backend>::set_cursor(_local2_param0_helper1, 65535, 12336);
```

Response: The developer has not responded to these bugs so far since this library is public archived.

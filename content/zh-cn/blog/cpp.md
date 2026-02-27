
`__attribute__ ((__packed__))` 保证了里面不会填充，就是紧密排列的，不会为了字节对齐而填充

```cpp
struct ether_addr
{
  uint8_t ether_addr_octet[ETH_ALEN];
} __attribute__ ((__packed__));
```

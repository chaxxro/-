# 字节序转换

```cpp
#include <endian.h>

// host to net 本机字节序转换为网络字节序
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
// net to host 网络字节序转换为本机字节序
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);


uint16_t htobe16(uint16_t bits);
uint16_t htole16(uint16_t bits);
uint16_t be16toh(uint16_t bits);
uint16_t le16toh(uint16_t bits);

uint32_t htobe32(uint32_t bits);
uint32_t htole32(uint32_t bits);
uint32_t be32toh(uint32_t bits);
uint32_t le32toh(uint32_t bits);

uint16_t htobe64(uint64_t bits);
uint16_t htole64(uint64_t bits);
uint16_t be64toh(uint64_t bits);
uint16_t le64toh(uint64_t bits);
```
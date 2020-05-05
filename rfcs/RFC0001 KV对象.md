# RFC0003 KV对象

## 设计原理

KV对象用于描述Curdata中带有数字签名认证功能的对象描述功能。KV对象可以被序列化成二进制，可以序列化成json表示形式，也可以进行签名验证。

## 具体设计

KV对象可以可以被序列化成二进制格式，具体的KV对象需要实现如下的trait。

```rust
trait KVObject: Serialize + DeserializeDe {
    type Bytes: AsRef<[u8]>;
    
    type Keypair: Keypair;
    
    // 从Bytes反序列化
	fn from_bytes(bytes: &Self::Bytes) -> Result<()>;
    
    // 序列化成Bytes
    fn to_bytes() -> Result<Self::Bytes>;
    
    // 根据key读取值
    fn get_key(key: &str) -> Result<Self::Bytes>;
    
    // 根据key写取值
    fn set_key(key: &str, value: Self::Bytes) -> Result<()>;
    
    // 验证签名是否正确
    fn validate() -> Result<bool>;
}
```

数字签名相关的键值设置：

```rust
trait Keypair: Serialize + DeserializeDe {
    type Seed;
    
    type Secret;
    
    type Public;
    
    type Code;
    
    type Signature: Serialize + DeserializeDe + AsRef<[u8]>;
    
    fn generate<R: Rng>(rng: &mut R) -> Result<Self>;
    
    fn generate_from_seed(seed: &Self::Seed) -> Result<Self>;
    
    fn sign<H: Hasher, R: Rng>() -> Result<Signature>;
    
    fn verify<H: Hasher, R: Rng>(sig: &Self::Signature) -> Result<bool>;
}
```



### 值序列化

通过Serde库进行序列化与反序列化时，具体的值需要被序列化成可读的格式，例如`hex`。
# RFC0001 KV对象

## 设计原理

KV对象用于描述Curdata中带有数字签名认证功能的对象描述功能。KV对象可以被序列化成二进制，可以序列化成json表示形式，也可以进行签名验证。

## 具体设计

KV对象可以可以被序列化成二进制格式，具体的KV对象需要实现如下的trait。

```rust
pub trait KValueObject: Serialize + for<'de> Deserialize<'de> + AttrProxy {
    type Bytes: Debug + AsRef<[u8]>;

    type Signature: Serialize + for<'de> Deserialize<'de> + Bytes;

    type KeyPair: asymmetric_crypto::prelude::Keypair<Signature = Self::Signature>;

    type Certificate: asymmetric_crypto::prelude::Certificate<Signature = Self::Signature>;

    // 从Bytes反序列化
    fn from_bytes(bytes: &Self::Bytes) -> Result<Self, KVObjectError>;

    // 序列化成Bytes
    fn to_bytes(&mut self, keypair: &Self::KeyPair) -> Result<Self::Bytes, KVObjectError>;
}
```

数字签名相关的键值设置：

```rust
pub trait Keypair: Debug + Clone + Serialize + for<'de> Deserialize<'de> {
    type Seed;

    type Secret;

    type Public;

    type Code;

    type Signature: Serialize + for<'de> Deserialize<'de> + Bytes;

    type Certificate: Certificate;

    fn generate<R: RngCore>(rng: &mut R) -> Result<Self, CryptoError>;

    fn generate_from_seed(seed: Self::Seed) -> Result<Self, CryptoError>;

    fn sign<H: Default + Hasher<Output = [u8; 32]> + Hasher, R: RngCore>(
        &self,
        msg: &[u8],
        rng: &mut R,
    ) -> Result<Self::Signature, CryptoError>;

    fn get_certificate(&self) -> Self::Certificate;
}

pub trait Certificate: Serialize + for<'de> Deserialize<'de> + Bytes {
    type Signature;

    fn verify<H: Default + Hasher<Output = [u8; 32]> + Hasher>(
        &self,
        msg: &[u8],
        signature: &Self::Signature,
    ) -> bool;
}

```



### 值序列化

通过Serde库进行序列化与反序列化时，具体的值需要被序列化成可读的格式，例如`hex`。
---
title: PHP和Go之AES的CBC模式加解密
date: 2024-09-29 16:15:52
tags: [php, Go]
categories: [php, Go]
---
### padding的填充方式可以根据自己需要修改
### php
```
$key       = 'a7gE3fH9jKmN1pQ2rS4tU6vY8zW9xL0';
$iv        = '7hJ3kQxZW45mNpR';
$data      = '123456';
$encrypted = openssl_encrypt($data, 'AES-256-CBC', $key, OPENSSL_RAW_DATA, $iv);
echo base64_encode($encrypted), PHP_EOL;
$decrypted = openssl_decrypt($encrypted, 'AES-256-CBC', $key, OPENSSL_RAW_DATA, $iv);
echo $decrypted;
```
### go
```
import (
	"bytes"
	"crypto/aes"
	"crypto/cipher"
	"crypto/md5"
	"encoding/base64"
	"encoding/json"
	"fmt"
)

func PKCS7Padding(ciphertext []byte, blockSize int) []byte {
	padding := blockSize - len(ciphertext)%blockSize
	padText := bytes.Repeat([]byte{byte(padding)}, padding)
	return append(ciphertext, padText...)
}

func PKCS7UnPadding(origData []byte) []byte {
	length := len(origData)
	unpadding := int(origData[length-1])
	return origData[:(length - unpadding)]
}

func AesEncryptCBC(orig string, key, iv string) string {
	origData := []byte(orig)
	k := []byte(key)

	block, _ := aes.NewCipher(k)

	blockSize := block.BlockSize()

	origData = PKCS7Padding(origData, blockSize)

	ivCopy := []byte(iv)
	blockMode := cipher.NewCBCEncrypter(block, ivCopy[:blockSize])

	cryted := make([]byte, len(origData))

	blockMode.CryptBlocks(cryted, origData)
	return base64.StdEncoding.EncodeToString(cryted)
}

func AesDecryptCBC(cryted string, key, iv string) string {
	crytedByte, _ := base64.StdEncoding.DecodeString(cryted)
	k := []byte(key)

	block, _ := aes.NewCipher(k)

	blockSize := block.BlockSize()

	ivCopy := []byte(iv)
	blockMode := cipher.NewCBCDecrypter(block, ivCopy[:blockSize])

	orig := make([]byte, len(crytedByte))

	blockMode.CryptBlocks(orig, crytedByte)

	orig = PKCS7UnPadding(orig)
	return string(orig)
}

func main() {
	key := "a7gE3fH9jKmN1pQ2rS4tU6vY8zW9xL0"
	iv := "7hJ3kQxZW45mNpR"
	data := "123456"
	encrypted := AesEncryptCBC(data, key, iv)
	fmt.Println(encrypted)
	decrypted := AesDecryptCBC(encrypted, key, iv)
	fmt.Println(decrypted)
}
```

# Operator:LOGADDEXP
# 1. Op Semantics
This article mainly explains the operator implementation of Logaddexp.
It calculates the logarithm of the sum of exponentiations of the inputs.
.
The implementation of the kernel is as follows:-

```bash
logaddexp(float dst, float lhs, float* rhs, unsigned int size);
```

| Arguments | Type     | Sementics                |
| :-------- | :------- | :------------------------- |
| `dst` | ` Vector type pointer` | Points to the output vector. |
| `lhs` | ` Vector type pointer` | It points the first input vector.. |
| `rhs` | `Vector type pointert` |  It points the second input vector.. |
| `size` | `Int64 `              | The number of elements in the Vector.|  

Example
```bash
    lhs ->⁡[-100.0, -200, -300]
	rhs ->  [-1.0, -2, -3]
	dst -> [-1., -2., -3.]
	size -> 3
```


# 2. Feature Specification

The implementation characteristics of this operator are described as follows:-

- Supported data types are PRED, INT8, UINT8, INT16, INT32, UINT32, FP16, BF16, FP32.
- Supported for scorpio (I30).
- Operands is supported in any dimension. *
- dtu implementation, pull mode. *




# 3. Design
![Screenshot](https://github.com/allen1245456/external_documentation/blob/main/Screenshot%202023-11-21%20142309.png)

#### Loading Vectors from CPU to Device Memory:
The code begins by transferring two vectors, host_vlhs and host_vrhs, from the CPU to the device memory (dev_vlhs and dev_vrhs) using memory copy operations (topsMemcpy). This step ensures that the data is available on the device for further processing.

#### Element-wise Exponentiation of Two Vectors:
After loading the vectors to device memory, the code performs element-wise exponentiation of the corresponding elements in dev_vlhs and dev_vrhs. This results in a new vector, dev_out, where each element is the exponentiation of the corresponding elements from the input vectors.

#### Calculating the Addition of Values:
The code then calculates the sum of values in the dev_out vector. This step involves adding up all the elements in the vector, potentially using a reduction operation. The result, sum_of_values, represents the sum of the exponentiated values.

#### Finding the Logarithm:
Finally, the code takes the natural logarithm (base e) of the sum of values (sum_of_values). The result, stored in the variable result_log, represents the logarithm of the sum of exponentiated values.




CPU Implementataion

```bash
template <typename TIN, typename TOUT>
void BinaryOpRef(TOUT *out, TIN vlhs, TIN vrhs, binaryOpType_t op) {
  switch (op) {
  // basic
  case LOGADDEXP:
      *out = (TOUT)std::log(std::exp(vlhs) + std::exp(vrhs));
    break;
  default:
    EXPECT_TRUE(0) << "unsupported op type";
    break;
  }
}
```

Host to HBM memcpy

```bash
 TOPS_CHECK(
        topsMemcpy(dev_vlhs, host_vlhs.data(), data_size, topsMemcpyHostToDevice));
    TOPS_CHECK(topsMemcpy(dev_vrhs, host_vrhs.data(), data_size,
                          topsMemcpyHostToDevice));
    TOPS_CHECK(topsMemset(dev_out, 0xff, data_size));

```

L3 to L1 memcpy

```bash

  tops::memcpy(ctx, tops::mdspan(tops::Private, local_vlhs, num),
               tops::mdspan(tops::Global, global_vlhs, num));

  tops::memcpy(ctx, tops::mdspan(tops::Private, local_vrhs, num),
               tops::mdspan(tops::Global, global_vrhs, num));
```




## 4. Pytorch Reference

[Pytorch Reference](https://pytorch.org/docs/stable/generated/torch.logaddexp.html)


## 5. Test
| Serial Number | Test Content     | Test Result                |
| :-------- | :------- | :------------------------- |
| `1` | `Data Type FP32` | -------- |
| `2` | `Data Type I16` | -------- |
| `3` | `Data Type I8` | -------- |








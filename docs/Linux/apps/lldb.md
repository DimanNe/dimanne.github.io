title: lldb

# **lldb**

## **Breakpoints**

Automatically perform an action on hit:

```lldb
breakpoint command add 1.1
> thread backtrace
> frame variable
> process continue
> DONE
```

or via a one-lineer syntax:
```
breakpoint command add 1.1 -o "bt"
```

## **Python scripting in lldb**

[Tutorial](https://lldb.llvm.org/use/python.html), [Reference 1](https://lldb.llvm.org/python_reference/), 
[Reference 2](https://lldb.llvm.org/python_api/), [Python enums](https://lldb.llvm.org/python_api_enums.html).


### List non-nullptr elements of an array

```python
script
inner = lldb.frame.FindVariable("alienInner")
buckets = inner.GetChildMemberWithName("mBuckets")
count = inner.GetChildMemberWithName("mBucketCount")

for i in range(count.signed):
   if buckets.GetChildAtIndex(i, lldb.eNoDynamicValues, True).unsigned != 0:
      print(f'{buckets.GetChildAtIndex(i, lldb.eNoDynamicValues, True)}')
```

### Use script from files

Create script:
```python title="parray.py"
import lldb
import shlex

def parray(debugger, command, result, dict):
    args = shlex.split(command)
    va = lldb.frame.FindVariable(args[0])
    for i in range(0, int(args[1])):
        print va.GetChildAtIndex(i, 0, 1)
```

Define a command `parray` in lldb:

```
(lldb) command script import /path/to/parray.py
(lldb) command script add --function parray.parray parray
```

Now you can use `parray` variable length:

```
(lldb) parray a 5
(double) *a = 0
(double) [1] = 0
(double) [2] = 1.14468
(double) [3] = 2.28936
(double) [4] = 3.43404
```

### Another example of scripting

```python
script


def PackedRow_to_str(row: lldb.SBValue):
  TAG_BIT                 = 1 << 0;
  BUCKET_TAIL_PTR_BIT     = 1 << 3;
  BUCKET_HEAD_PTR_BIT     = 1 << 4;
  FINAL_BIT               = 1 << 5;
  END_OF_BUCKET_BIT       = 1 << 6;
  NEXT_ROW_PACKET_PTR_BIT = 1 << 7;
  
  m_flags: int = row.GetChildMemberWithName("mFlags").unsigned
  result: str = f'len: {row.GetChildMemberWithName("mLength").unsigned}'

  if m_flags & BUCKET_TAIL_PTR_BIT:
    result += " tail"
  if m_flags & BUCKET_HEAD_PTR_BIT:
    result += " head"
  if m_flags & FINAL_BIT:
    result += " final"
  if m_flags & NEXT_ROW_PACKET_PTR_BIT:
    result += " ptr"

  return result


def print_ResizingHash_as(inner: lldb.SBValue, value_name: str = "PackedRow", max_count: int = None):
  target = lldb.debugger.GetSelectedTarget()
  packed_row_type = target.FindFirstType(value_name)
  BLOOM_MASK = 0xFFFF000000000000
  PTR_MASK = ~BLOOM_MASK
  bucket_count: lldb.SBValue = inner.GetChildMemberWithName("mBucketCount")
  for i in range(min(bucket_count.signed, max_count) if max_count else bucket_count.signed):
    bucket = buckets.GetChildAtIndex(i, lldb.eNoDynamicValues, True)
    if bucket.unsigned == 0:
        continue
    bloom_val = (bucket.GetValueAsUnsigned() & BLOOM_MASK) >> 48
    ptr_val = bucket.GetValueAsUnsigned() & PTR_MASK
    packed_row: lldb.SBValue = target.CreateValueFromAddress(f"i={i}", lldb.SBAddress(ptr_val, target), packed_row_type)
    print(f'{packed_row.AddressOf()}, bloom: {bloom_val}, {value_name}: {PackedRow_to_str(packed_row)}')





inner = lldb.frame.FindVariable("srcInner")
print_ResizingHash_as(inner, "PackedRow", 100)
(PackedRow *) &i=61 = 0x00007fff9e13a164, bloom: 1125899906842624, PackedRow: len: 0
(PackedRow *) &i=62 = 0x00007fff9e1202c8, bloom: 281474976710656, PackedRow: len: 0
(PackedRow *) &i=986 = 0x00007fff9e120a48, bloom: 2, PackedRow: len: 2304
(PackedRow *) &i=991 = 0x00007fff9e125a4c, bloom: 2, PackedRow: len: 23056 tail head
(PackedRow *) &i=998 = 0x00007fff9e10c8ac, bloom: 4, PackedRow: len: 34304 tail head final ptr
(PackedRow *) &i=999 = 0x00007fff9e12d27c, bloom: 8, PackedRow: len: 48 tail
```

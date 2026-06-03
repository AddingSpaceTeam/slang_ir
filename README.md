# slang_ir
Main goal of `slang_ir` is to provide access to the internal intermediate representation of slang. This will help creating custom shader languages compilers with leveraging the backends available in slang, automatic differentiation (great for ml) and slang's optimization capabilities.

API design of this library is heavily inspired by `libgccjit`.

# Types
```c
type slang_ir_type
type slang_ir_vector_type
type slang_ir_array_type
type slang_ir_matrix_type
```
## Basic types
```c
// get base type
slang_ir_type *slang_ir_type_get(slang_ir_builder* builder, enum slang_ir_types type_)

enum
slang_ir_types
{
  SLANG_IR_TYPE_VOID,
  SLANG_IR_TYPE_BOOL,
  SLANG_IR_TYPE_INT8,
  SLANG_IR_TYPE_INT16,
  SLANG_IR_TYPE_INT,
  SLANG_IR_TYPE_INT64,
  SLANG_IR_TYPE_UINT8,
  SLANG_IR_TYPE_UINT16,
  SLANG_IR_TYPE_UINT,
  SLANG_IR_TYPE_UINT64,
  SLANG_IR_TYPE_HALF,
  SLANG_IR_TYPE_FLOAT,
  SLANG_IR_TYPE_DOUBLE,
};

// get int type with specified size
slang_ir_type *slang_ir_type_get_int(slang_ir_builder* b, int num_bytes, int is_signed)
```
## Vector types
```lua
Vec = {
					struct_name = "VectorType",
					operands = { { "elementType", "IRType" }, { "elementCount" } },
					hoistable = true,
      },
```
```c
// get vec type
slang_ir_vector_type *slang_ir_type_get_vector(slang_ir_type *type, size_t num_elements)

size_t slang_ir_vector_type_get_num_elements (slang_ir_vector_type *vector_type);

gcc_jit_type *slang_ir_vector_type_get_element_type (slang_ir_vector_type *vector_type);

// sizeof
size_t slang_ir_type_get_size(slang_ir_type *type)
```
## Array types
```lua
ArrayTypeBase = {
    hoistable = true,
    {
        Array = {
            struct_name = "ArrayType",
            operands = {
                { "elementType", "IRType" },
                { "elementCount" },
                { "stride", optional = true },
            },
        },
    },
    {
        UnsizedArray = {
            struct_name = "UnsizedArrayType",
            operands = { { "elementType", "IRType" }, { "stride", optional = true } },
        },
    },
},
```
```c
slang_ir_array_type *slang_ir_type_get_array(slang_ir_type *type, int64_t num_elements, int stride);
slang_ir_array_type *slang_ir_type_get_unsized_array(slang_ir_type *type, int stride);
```
## Matrix types
```lua
Mat = {
					struct_name = "MatrixType",
					operands = { { "elementType", "IRType" }, { "rowCount" }, { "columnCount" }, { "layout" } },
					hoistable = true,
      },
```
```c
enum
slang_ir_matrix_layouts
{
  SLANG_IR_MATRIX_LAYOUT_UNKNOWN     = 0,
  SLANG_IR_MATRIX_LAYOUT_ROW_MAJOR   = 1,
  SLANG_IR_MATRIX_LAYOUT_COLUMN_MAJOR = 2,
};

slang_ir_matrix_type slang_ir_type_get_matrix(slang_ir_type *type, int64_t num_rows, int64_t num_cols, enum slang_ir_matrix_layouts layout)
```

`slang-ir-insts.lua` have many types that better to develop as builtin (or magic) types. 

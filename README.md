# slang_ir
Main goal of `slang_ir` is to provide access to the internal intermediate representation of slang. This will help creating custom shader languages compilers with leveraging the backends available in slang, automatic differentiation (great for ml) and slang's optimization capabilities.

API design of this library is heavily inspired by `libgccjit`.

Library targets to mid-level instructions (not high-level that slang specific like Result type), but with exceptions for automatic differentiation support.

# Locations
```c
// source code location
type slang_ir_location
```
you can always pass NULL to any API entrypoint accepting one.
```c
slang_ir_location_create(slang_ir_builder* builder, const char* filename, int line, int column)
```

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
## Functions
Unlike libgccjit, we need to create function type and then function, it better map to modern programming languages.
First param is return type / return param
```
slang_ir_function_param_create(slang_ir_builder* builder, slang_ir_location* locs, lang_ir_type* type, const char* name)

slang_ir_type_get_func(int num_params, gcc_jit_param **params)

slang_ir_function slang_ir_function_create(slang_ir_builder* builder, slang_ir_type* function_type, const char *name)
```
## Function annotations
### Entry point
```c
enum
slang_ir_entry_point_annotations
{
  SLANG_IR_ENTRY_POINT_ANNOTATION_STAGE_VERTEX,
  SLANG_IR_ENTRY_POINT_ANNOTATION_STAGE_FRAGMENT,
  SLANG_IR_ENTRY_POINT_ANNOTATION_STAGE_COMPUTE,
  // place for tesselation stages etc.
}

void slang_ir_func_add_entry_point_annotation(slang_ir_builder* builder, slang_ir_function* func, enum slang_ir_entry_point_annotations annotation)
```
### Numthreads
```c
void slang_ir_func_add_numthreads_annotation(slang_ir_builder* builder, slang_ir_function* func, int x, int y, int z)
```

### Auto differentiation
API currently support annotations only for normal differentiation, without custom differentiation support in user code. For it, these 3 annotations is enough
```c
enum
slang_ir_differentiation_annotations
{
  SLANG_IR_DIFFERENTIATION_ANNOTATION_ZERO,
  SLANG_IR_DIFFERENTIATION_ANNOTATION_TYPE,
  SLANG_IR_DIFFERENTIATION_ANNOTATION_ADD
}

slang_ir_function slang_ir_differentiate(slang_ir_builder* builder, slang_ir_function* func)
void slang_ir_add_differentiation_annotation(slang_ir_builder* builder, slang_ir_function* func, enum slang_ir_differentiation_annotations annotation)
```

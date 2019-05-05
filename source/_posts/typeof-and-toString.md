---
title: typeof和toString的差别
date: 2019-05-05 14:45:00
tags: [javascript]
---
以下参考`jerryscript`项目中关于 **ecma** 的实现：

`typeof`是直接判断值类型(位于`jerry-core/vm/opcodes.c`)：
```c
/**
 * 'typeof' opcode handler.
 *
 * See also: ECMA-262 v5, 11.4.3
 *
 * @return ecma value
 *         Returned value must be freed with ecma_free_value
 */
ecma_value_t
opfunc_typeof (ecma_value_t left_value) /**< left value */
{
  ecma_value_t ret_value = ecma_make_simple_value (ECMA_SIMPLE_VALUE_EMPTY);

  ecma_string_t *type_str_p = NULL;

  if (ecma_is_value_undefined (left_value))
  {
    type_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_UNDEFINED);
  }
  else if (ecma_is_value_null (left_value))
  {
    type_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_OBJECT);
  }
  else if (ecma_is_value_boolean (left_value))
  {
    type_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_BOOLEAN);
  }
  else if (ecma_is_value_number (left_value))
  {
    type_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_NUMBER);
  }
  else if (ecma_is_value_string (left_value))
  {
    type_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_STRING);
  }
  else
  {
    JERRY_ASSERT (ecma_is_value_object (left_value));

    if (ecma_op_is_callable (left_value))
    {
      type_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_FUNCTION);
    }
    else
    {
      type_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_OBJECT);
    }
  }

  ret_value = ecma_make_string_value (type_str_p);

  return ret_value;
} /* opfunc_typeof */
```
而Object.prototype.toString是泛型方法，只判断类类型(位于`jerry-core/emca/builtin-objects/ecma-builtin-object-prototype.c`)：
```c
/**
 * The Object.prototype object's 'toString' routine
 *
 * See also:
 *          ECMA-262 v5, 15.2.4.2
 *
 * @return ecma value
 *         Returned value must be freed with ecma_free_value.
 */
static ecma_value_t
ecma_builtin_object_prototype_object_to_string (ecma_value_t this_arg) /**< this argument */
{
  return ecma_builtin_helper_object_to_string (this_arg);
} /* ecma_builtin_object_prototype_object_to_string */
```

```c
/** \addtogroup ecma ECMA
 * @{
 *
 * \addtogroup ecmabuiltinhelpers ECMA builtin helper operations
 * @{
 */

/**
 * Common implementation of the Object.prototype.toString routine
 *
 * See also:
 *          ECMA-262 v5, 15.2.4.2
 *
 * Used by:
 *         - The Object.prototype.toString routine.
 *         - The Array.prototype.toString routine as fallback.
 *
 * @return ecma value
 *         Returned value must be freed with ecma_free_value.
 */

ecma_value_t
ecma_builtin_helper_object_to_string (const ecma_value_t this_arg) /**< this argument */
{
  lit_magic_string_id_t type_string;

  if (ecma_is_value_undefined (this_arg))
  {
    type_string = LIT_MAGIC_STRING_UNDEFINED_UL;
  }
  else if (ecma_is_value_null (this_arg))
  {
    type_string = LIT_MAGIC_STRING_NULL_UL;
  }
  else
  {
    ecma_value_t obj_this = ecma_op_to_object (this_arg);

    if (ECMA_IS_VALUE_ERROR (obj_this))
    {
      return obj_this;
    }

    JERRY_ASSERT (ecma_is_value_object (obj_this));

    ecma_object_t *obj_p = ecma_get_object_from_value (obj_this);

    type_string = ecma_object_get_class_name (obj_p);

    ecma_free_value (obj_this);
  }

  ecma_string_t *ret_string_p;

  /* Building string "[object #type#]" where type is 'Undefined',
     'Null' or one of possible object's classes.
     The string with null character is maximum 27 characters long. */
  const lit_utf8_size_t buffer_size = 27;
  lit_utf8_byte_t str_buffer[buffer_size];

  lit_utf8_byte_t *buffer_ptr = str_buffer;

  const lit_magic_string_id_t magic_string_ids[] =
  {
    LIT_MAGIC_STRING_LEFT_SQUARE_CHAR,
    LIT_MAGIC_STRING_OBJECT,
    LIT_MAGIC_STRING_SPACE_CHAR,
    type_string,
    LIT_MAGIC_STRING_RIGHT_SQUARE_CHAR
  };

  for (uint32_t i = 0; i < sizeof (magic_string_ids) / sizeof (lit_magic_string_id_t); ++i)
  {
    buffer_ptr = lit_copy_magic_string_to_buffer (magic_string_ids[i], buffer_ptr,
                                                  (lit_utf8_size_t) ((str_buffer + buffer_size) - buffer_ptr));
    JERRY_ASSERT (buffer_ptr <= str_buffer + buffer_size);
  }

  ret_string_p = ecma_new_ecma_string_from_utf8 (str_buffer, (lit_utf8_size_t) (buffer_ptr - str_buffer));

  return ecma_make_string_value (ret_string_p);
} /* ecma_builtin_helper_object_to_string */
```

```c
lit_magic_string_id_t
ecma_object_get_class_name (ecma_object_t *obj_p) /**< object */
{
  ecma_object_type_t type = ecma_get_object_type (obj_p);

  switch (type)
  {
    case ECMA_OBJECT_TYPE_ARRAY:
    {
      return LIT_MAGIC_STRING_ARRAY_UL;
    }
    case ECMA_OBJECT_TYPE_CLASS:
    {
      ecma_extended_object_t *ext_object_p = (ecma_extended_object_t *) obj_p;
      return ext_object_p->u.class_prop.class_id;
    }
    case ECMA_OBJECT_TYPE_PSEUDO_ARRAY:
    {
      ecma_extended_object_t *ext_obj_p = (ecma_extended_object_t *) obj_p;

      switch (ext_obj_p->u.pseudo_array.type)
      {
        case ECMA_PSEUDO_ARRAY_ARGUMENTS:
        {
          return LIT_MAGIC_STRING_ARGUMENTS_UL;
        }
#ifndef CONFIG_DISABLE_ES2015_TYPEDARRAY_BUILTIN
        case ECMA_PSEUDO_ARRAY_TYPEDARRAY:
        case ECMA_PSEUDO_ARRAY_TYPEDARRAY_WITH_INFO:
        {
          return ext_obj_p->u.pseudo_array.u1.class_id;
        }
#endif /* !CONFIG_DISABLE_ES2015_TYPEDARRAY_BUILTIN */
        default:
        {
          JERRY_UNREACHABLE ();
        }
      }

      JERRY_UNREACHABLE ();
      break;
    }
    case ECMA_OBJECT_TYPE_FUNCTION:
    case ECMA_OBJECT_TYPE_EXTERNAL_FUNCTION:
    case ECMA_OBJECT_TYPE_BOUND_FUNCTION:
    {
      return LIT_MAGIC_STRING_FUNCTION_UL;
    }
    default:
    {
      JERRY_ASSERT (type == ECMA_OBJECT_TYPE_GENERAL);

      if (ecma_get_object_is_builtin (obj_p))
      {
        ecma_extended_object_t *ext_obj_p = (ecma_extended_object_t *) obj_p;

        switch (ext_obj_p->u.built_in.id)
        {
#ifndef CONFIG_DISABLE_MATH_BUILTIN
          case ECMA_BUILTIN_ID_MATH:
          {
            return LIT_MAGIC_STRING_MATH_UL;
          }
#endif /* !CONFIG_DISABLE_MATH_BUILTIN */
#ifndef CONFIG_DISABLE_JSON_BUILTIN
          case ECMA_BUILTIN_ID_JSON:
          {
            return LIT_MAGIC_STRING_JSON_U;
          }
#endif /* !CONFIG_DISABLE_JSON_BUILTIN */
#ifndef CONFIG_DISABLE_ERROR_BUILTINS
          case ECMA_BUILTIN_ID_EVAL_ERROR_PROTOTYPE:
          case ECMA_BUILTIN_ID_RANGE_ERROR_PROTOTYPE:
          case ECMA_BUILTIN_ID_REFERENCE_ERROR_PROTOTYPE:
          case ECMA_BUILTIN_ID_SYNTAX_ERROR_PROTOTYPE:
          case ECMA_BUILTIN_ID_TYPE_ERROR_PROTOTYPE:
          case ECMA_BUILTIN_ID_URI_ERROR_PROTOTYPE:
#endif /* !CONFIG_DISABLE_ERROR_BUILTINS */
          case ECMA_BUILTIN_ID_ERROR_PROTOTYPE:
          {
            return LIT_MAGIC_STRING_ERROR_UL;
          }
          default:
          {
            JERRY_ASSERT (ecma_object_check_class_name_is_object (obj_p));

            return LIT_MAGIC_STRING_OBJECT_UL;
          }
        }
      }
      else
      {
        return LIT_MAGIC_STRING_OBJECT_UL;
      }
    }
  }
} /* ecma_object_get_class_name */
```
对于其它的具体类型，仍然是判断值，例如：
```
/** \addtogroup ecma ECMA
 * @{
 *
 * \addtogroup ecmabuiltins
 * @{
 *
 * \addtogroup booleanprototype ECMA Boolean.prototype object built-in
 * @{
 */

/**
 * The Boolean.prototype object's 'toString' routine
 *
 * See also:
 *          ECMA-262 v5, 15.6.4.2
 *
 * @return ecma value
 *         Returned value must be freed with ecma_free_value.
 */
static ecma_value_t
ecma_builtin_boolean_prototype_object_to_string (ecma_value_t this_arg) /**< this argument */
{
  ecma_value_t ret_value = ecma_make_simple_value (ECMA_SIMPLE_VALUE_EMPTY);

  ECMA_TRY_CATCH (value_of_ret,
                  ecma_builtin_boolean_prototype_object_value_of (this_arg),
                  ret_value);

  ecma_string_t *ret_str_p;

  if (ecma_is_value_true (value_of_ret))
  {
    ret_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_TRUE);
  }
  else
  {
    JERRY_ASSERT (ecma_is_value_boolean (value_of_ret));

    ret_str_p = ecma_get_magic_string (LIT_MAGIC_STRING_FALSE);
  }

  ret_value = ecma_make_string_value (ret_str_p);

  ECMA_FINALIZE (value_of_ret);

  return ret_value;
} /* ecma_builtin_boolean_prototype_object_to_string */
```
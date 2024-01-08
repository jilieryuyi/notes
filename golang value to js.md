golang wasm 

convert any

convert struct to js valid(object) value

convert map to js valid(object) value

convert array to js valid(object) value

```

import (
	"fmt"
	"reflect"
)

func JsValidValue(data interface{}) interface{} {
	v := reflect.ValueOf(data)
	t := v.Kind()
	switch t {
	case reflect.Bool,
		reflect.Int,
		reflect.Int8,
		reflect.Int16,
		reflect.Int32,
		reflect.Int64,
		reflect.Uint,
		reflect.Uint8,
		reflect.Uint16,
		reflect.Uint32,
		reflect.Uint64,
		//Uintptr
		reflect.Float32,
		reflect.Float64,
		//Complex64
		//Complex128
		//Array
		//Chan
		//Func
		//Interface
		//Map
		//Pointer
		//Slice
		reflect.String:
		//Struct
		//UnsafePointer:
		return data
	case reflect.Map:
		var realValue = make(map[string]interface{})
		res := reflect.MakeMap(v.Type())
		keys := v.MapKeys()
		for _, k := range keys {
			k := k.Convert(res.Type().Key())
			key := k.Interface()
			value := v.MapIndex(k).Interface()

			realValue[fmt.Sprintf("%v", key)] = JsValidValue(value)
		}
		return realValue
	case reflect.Pointer:
		if v.IsZero() {
			return nil
		}

		return JsValidValue(v.Elem().Interface())
	case reflect.Slice, reflect.Array:
		var arr []interface{}
		for i := 0; i < v.Len(); i++ {
			iv := v.Index(i)
			arr = append(arr, JsValidValue(iv.Interface()))
		}

		return arr
	case reflect.Struct:
		var realValue = make(map[string]interface{})
		for i := 0; i < v.NumField(); i++ {
			tf := v.Type().Field(i)

			tag := tf.Tag.Get("js")
			if tag == "" {
				tag = tf.Tag.Get("json")
			}
			if tag == "" {
				tag = tf.Name
			}

			realValue[tag] = JsValidValue(v.Field(i).Interface())
		}

		return realValue
	}

	return -1
}

```

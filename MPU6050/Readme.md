# Uso de la MPU6050(GY-521)

1. Vemos si está vivo el sensor. Lo he cogado del segundo bus I2C y lo he configurado en el cube como `FastMode`

```c 
 
  HAL_I2C_Mem_Read (&hi2c2, 0xD0 ,0x75,1, &check, 1, 1000);
  //debe responder 104 en decimal que es 0x68. 0xD0 es la dirección y el registro 0x75 devuelve 0x68 si todo va bien.
```
2. Ahora despertamos al sensor escribiendo `0x00`en el registro `0x6B`

```c 
Data = 0;
HAL_I2C_Mem_Write(&hi2c2, 0xD0, 0x6B, 1,&Data, 1, 1000);
```
3.Ahora tenemos que configurar la frecuencia de salida de datos o la frecuencia de muestreo. Esto se puede hacer escribiendo en el registro `“SMPLRT_DIV (0x19)”`. Este registro especifica el divisor de la tasa de salida del giroscopio que se usa para generar la frecuencia de muestreo para el `MPU6050`.
fórmula `Tasa de muestreo = Tasa de salida del giroscopio / (1 + SMPLRT_DIV)`. Donde la frecuencia de salida del giroscopio es de` 8 KHz`, para obtener la frecuencia de muestreo de `1 KHz`, necesitamos usar el `SMPLRT_DIV como "7"`.

```c
Data = 0x07;
HAL_I2C_Mem_Write(&hi2c2, 0xD0, 0x19, 1, &Data, 1, 1000);
```
4. Ahora configure los registros del Acelerómetro y del Giroscopio y para ello necesitamos modificar los Registros `“GYRO_CONFIG (0x1B)” y “ACCEL_CONFIG (0x1C)”`.
Escribir `(0x00)` en ambos registros establecería el rango de escala completa de` ± 2g en ACCEL_CONFIGRegister `y un rango de escala completa de `± 250 ° / s en GYRO_CONFIGRegister` junto con la autoprueba desactivada.
```c 
Data = 0;
HAL_I2C_Mem_Write(&hi2c2, 0xD0, 0x1B, 1,&Data, 1, 1000);
HAL_I2C_Mem_Write(&hi2c2, 0xD0, 0x1C, 1,&Data, 1, 1000);
```
5.Podemos leer 1 BYTE de cada Registro por separado o podemos leer 6 BYTES todos juntos comenzando desde el Registro` ACCEL_XOUT_H`.
El registro `ACCEL_XOUT_H (0x3B) `almacena el byte más alto para los datos de aceleración a lo largo del eje X y el byte inferior se almacena en el registro `ACCEL_XOUT_L`. Entonces, necesitamos combinar estos `2 BYTES` en un valor entero de `16 bits`. A continuación se muestra el proceso para hacer eso:
```c
ACCEL_X = (ACCEL_XOUT_H <<8 | ACCEL_XOUT_L)
```
`ACCEL_XOUT_H = 11101110 y ACCEL_XOUT_L = 10101010, obtendremos el valor de 16 bits resultante como 1110111010101010`

6. De manera similar, podemos hacer lo mismo para los registros `ACCEL_YOUT y ACCEL_ZOUT`. Estos valores seguirán siendo los valores RAW y todavía debemos convertirlos al formato "g" adecuado.
`para el rango de escala completa de ± 2g, la sensibilidad es 16384 LSB / g. Entonces, para obtener el valor "g", debemos dividir el RAW de 16384`
```c
uint8_t Rec_Data[6];
HAL_I2C_Mem_Read (&hi2c2, 0x0D, 0x3B, 1, Rec_Data, 6, 1000);

Accel_X_RAW = (int16_t)(Rec_Data[0] << 8 | Rec_Data [1]);
Accel_Y_RAW = (int16_t)(Rec_Data[2] << 8 | Rec_Data [3]);
Accel_Z_RAW = (int16_t)(Rec_Data[4] << 8 | Rec_Data [5]);
Ax = Accel_X_RAW/16384.0;
Ay = Accel_Y_RAW/16384.0;
Az = Accel_Z_RAW/16384.0;
```
7. La lectura de Gyro Data es similar al caso de Aceleración. Comenzaremos a leer `6 BYTES` de datos del Registro ` GYRO_XOUT_H`. Combino los 2 Bytes para obtener valores RAW enteros de 16 bits. Como hemos seleccionado el rango de escala completa de `± 250 ° / s`, para el cual la sensibilidad es `131 LSB / ° / s`, tenemos que dividir los valores RAW por `131.0` para obtener los valores en `dps (° / s)`.
```c
HAL_I2C_Mem_Read (&hi2c2, 0x0D, 0x43, 1, Rec_Data, 6, 1000);

Gyro_X_RAW = (int16_t)(Rec_Data[0] << 8 | Rec_Data [1]);
Gyro_Y_RAW = (int16_t)(Rec_Data[2] << 8 | Rec_Data [3]);
Gyro_Z_RAW = (int16_t)(Rec_Data[4] << 8 | Rec_Data [5]);
Gx = Gyro_X_RAW/131.0;
Gy = Gyro_Y_RAW/131.0;
Gz = Gyro_Z_RAW/131.0;
```

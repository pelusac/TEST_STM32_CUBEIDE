## Uso de la librería más ligera para el la pantalla OLED basada en`ssd1306`

1. Agregar al proyecto bajo la carpeta `drivers`el directorio completo `ssd_1306` y agregarlo al `PATH`con botón derecho sobre los directorios.
2. En la configuración del `bus I2C`no olvidar pornerlo en `fastMode`
3. Incluir en el `main.c`
```c 
#include "ssd1306.h"
#include "fonts.h"
#include "test.h"
```
4. Abrir el fichero `ssd1306.c`y cambiar si corresponde ```extern I2C_HandleTypeDef hi2c1;```
5. Dependiendo del micro que usemos el compilador se quejará en los siguientes ficheros `fonts.h` y `ssd1306.h`donde habrá que cambiar
la línea ` #include "stm32f1xx_hal.h"`por el `stm32f`que corresponda en mi caso el `f0`. `#include "stm32f0xx_hal.h"`
6. Agregamos funcines para testear la pantalla.
```c 
/* USER CODE BEGIN 2 */
  SSD1306_Init (); // initialize the display

  SSD1306_GotoXY (10,10); // goto 10, 10
  SSD1306_Puts ("HOLA", &Font_11x18, 1); // print Hello
  SSD1306_GotoXY (10, 30);
  SSD1306_Puts ("STM32F0", &Font_11x18, 1);
  SSD1306_UpdateScreen(); // update screen
  /* USER CODE END 2 */
```
4. En el fichero `ssd1306.h` se puede definir otra dirección de dispositivo.
```c 
/* I2C address */
#ifndef SSD1306_I2C_ADDR
#define SSD1306_I2C_ADDR         0x78
//#define SSD1306_I2C_ADDR       0x7A
#endif
```
5. Función para convertir variables e imprimirlas en la pantalla
```c
 timer = __HAL_TIM_GET_COUNTER(&htim3);
	  sprintf(MSG, "Ticks = %ld",timer);
	  SSD1306_GotoXY (10,10);
	  SSD1306_Puts (MSG, &Font_7x10, 1);
	  SSD1306_UpdateScreen();
	  //HAL_Delay(100);
	  HAL_UART_Transmit(&huart2, (uint16_t*)MSG, sizeof(MSG), 100);
	  SSD1306_Puts ("", &Font_7x10, 1);
    ```

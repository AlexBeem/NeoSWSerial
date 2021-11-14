#
Класс NeoSWSerial предназначен для более эффективной замены встроенного программного обеспечения класса Arduino. Если вы могли бы использовать Serial, Serial1, Serial2 или Serial3, вам следует использовать вместо этого NeoHWSerial. Если бы вы могли использовать входной вывод захвата (ICP1, контакты 8 и 9 на UNO), вам следует вместо этого рассмотреть NeoICSerial.

NeoSWSerial ограничен четырьмя скоростями передачи данных: 9600 (по умолчанию), 19200, 31250 (MIDI) и 38400.

Методы класса почти идентичны встроенному программному обеспечению, за исключением двух новых методов, attachInterrupt и detachInterrupt:

```
    typedef void (* isr_t)( uint8_t );
    void attachInterrupt( isr_t fn );
    void detachInterrupt() { attachInterrupt( (isr_t) NULL ); };

  private:
    isr_t  _isr;
```

Есть пять, нет, шесть преимуществ перед программным обеспечением.:

1) Он использует гораздо меньше процессорного времени.

2) Одновременная передача и прием полностью поддерживаются.

3) Прерывания не отключаются в течение всего времени действия символа RX. (Они отключены в течение большей части времени каждого символа TX.)

4) Это намного надежнее (гораздо меньше ошибок при получении данных).

5) Символы могут обрабатываться с помощью определенной пользователем процедуры во время прерывания. Это должно предотвратить большинство проблем с переполнением входного буфера. Просто зарегистрируйте свою процедуру в экземпляре "NeoSWSerial":

```
    #include <NeoSWSerial.h>
    NeoSWSerial ss( 4, 3 );
    
    volatile uint32_t newlines = 0UL;
    
    static void handleRxChar( uint8_t c )
    {
      if (c == '\n')
        newlines++;
    }
    
    void setup()
    {
      ss.attachInterrupt( handleRxChar );
      ss.begin( 9600 );
    }
```

Помните, что зарегистрированная процедура вызывается из контекста прерывания, и она должна возвращаться как можно быстрее. Слишком много времени на выполнение процедуры приведет ко многим непредсказуемым последствиям, включая потерю полученных данных. Смотрите аналогичные предупреждения для встроенного устройства attachInterrupt для цифровых контактов.

Зарегистрированная процедура будет вызываться из ISR всякий раз, когда будет получен символ. Полученный символ не будет сохранен в rx_buffer, и он не будет возвращен из read(). Любые символы, которые были получены и буферизованы до вызова attachInterrupt, остаются в rx_buffer и могут быть извлечены путем вызова read().

Если attachInterrupt никогда не вызывается или передается процедура NULL, происходит обычная буферизация, и все полученные символы должны быть получены путем вызова read().

6) NeoSWSerial ISR могут быть отключены. Это может помочь вам избежать конфликтов при связывании с другими библиотеками PinChangeInterrupt, такими как enableinterrupt:

```
void myDeviceISR()
{
  NeoSWSerial::rxISR( *portInputRegister( digitalPinToPort( RX_PIN ) ) );
  // if you know the exact PIN register, you could do this:
  //    NeoSWSerial::rxISR( PIND );
}

void setup()
{
  myDevice.begin( 9600 );
  enableInterrupt( RX_PIN, myDeviceISR, CHANGE );
  enableInterrupt( OTHER_PIN, otherISR, RISING );
}
```

Этот класс поддерживает следующие микроконтроллеры: attinyx61, attinyx4, attinyx5, ATmega328P (Pro, UNO, Nano), ATmega32U4 (Micro, Leonardo), ATmega2560 (Mega), ATmega2560RFR2, ATmega1284P и atmega1286
#
#

The **NeoSWSerial** class is intended as an more-efficient drop-in replacement for the Arduino built-in class `SoftwareSerial`.  If you could use `Serial`, `Serial1`, `Serial2` or `Serial3`, you should use [NeoHWSerial](https://github.com/SlashDevin/NeoHWSerial) instead.  If you could use an Input Capture pin (ICP1, pins 8 & 9 on an UNO), you should consider  [NeoICSerial](https://github.com/SlashDevin/NeoICSerial) instead.

**NeoSWSerial** is limited to four baud rates: 9600 (default), 19200, 31250 (MIDI) and 38400.

The class methods are nearly identical to the built-in `SoftwareSerial`, except for two new methods, `attachInterrupt` and `detachInterrupt`:

```
    typedef void (* isr_t)( uint8_t );
    void attachInterrupt( isr_t fn );
    void detachInterrupt() { attachInterrupt( (isr_t) NULL ); };

  private:
    isr_t  _isr;
```

There are five, nay, **six** advantages over `SoftwareSerial`:

**1)** It uses *much* less CPU time.  

**2)** Simultaneous transmit and receive is fully supported.

**3)** Interrupts are not disabled for the entire RX character time.  (They are disabled for most of each TX character time.)

**4)** It is much more reliable (far fewer receive data errors).

**5)** Characters can be handled with a user-defined procedure at interrupt time.  This should prevent most input buffer overflow problems.  Simply register your procedure with the 'NeoSWSerial' instance:

```
    #include <NeoSWSerial.h>
    NeoSWSerial ss( 4, 3 );
    
    volatile uint32_t newlines = 0UL;
    
    static void handleRxChar( uint8_t c )
    {
      if (c == '\n')
        newlines++;
    }
    
    void setup()
    {
      ss.attachInterrupt( handleRxChar );
      ss.begin( 9600 );
    }
```

Remember that the registered procedure is called from an interrupt context, and it should return as quickly as possible.  Taking too much time in the procedure will cause many unpredictable behaviors, including loss of received data.  See the similar warnings for the built-in [`attachInterrupt`](https://www.arduino.cc/en/Reference/AttachInterrupt) for digital pins.

The registered procedure will be called from the ISR whenever a character is received.  The received character **will not** be stored in the `rx_buffer`, and it **will not** be returned from `read()`.  Any characters that were received and buffered before `attachInterrupt` was called remain in `rx_buffer`, and could be retrieved by calling `read()`.

If `attachInterrupt` is never called, or it is passed a `NULL` procedure, the normal buffering occurs, and all received characters must be obtained by calling `read()`.

**6)** The NeoSWSerial ISRs can be disabled.  This can help you avoid linking conflicts with other PinChangeInterrupt libraries, like EnableInterrupt:

```
void myDeviceISR()
{
  NeoSWSerial::rxISR( *portInputRegister( digitalPinToPort( RX_PIN ) ) );
  // if you know the exact PIN register, you could do this:
  //    NeoSWSerial::rxISR( PIND );
}

void setup()
{
  myDevice.begin( 9600 );
  enableInterrupt( RX_PIN, myDeviceISR, CHANGE );
  enableInterrupt( OTHER_PIN, otherISR, RISING );
}
```

This class supports the following MCUs: ATtinyx61, ATtinyx4, ATtinyx5, ATmega328P (Pro, UNO, Nano), ATmega32U4 (Micro, Leonardo), ATmega2560 (Mega), ATmega2560RFR2, ATmega1284P and ATmega1286

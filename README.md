# СКРИПТЫ с детальными комментариями на google colab:
<a href="https://colab.research.google.com/drive/1hY4sijr1Gw_4y67c38CbUiG5-dhzxmNf?usp=sharing" target="_blank"> Скрипты на питоне (ctrl + правая кнопка мышки) </a>

# Предоставляю отчет в md по выполненной работе
# Оглавление
1. [Введение](#1-введение)
   - Программно-Определяемое Радио (SDR)
   - Цифровая Обработка Сигналов (DSP)
2. [Частотная область](#2-частотная-область)
3. [Выборка IQ](#3-выборка-iq)
4. [Цифровая модуляция](#4-цифровая-модуляция)
5. [PlutoSDR в Python](#5-plutosdr-в-python)
6. [USRP в Python](#6-usrp-в-python)
7. [Шум и дБ](#7-шум-и-дб)
8. [Фильтры](#8-фильтры)
9. [Link budgets()](#9-link-budgets)
10. [Кодирование канала](#10-кодирование-канала)
11. [IQ файлы и SigMF](#11-iq-файлы-и-sigmf)
12. [Многолучевое затухание](#12-многолучевое-затухание)
13. [Формирование импульса](#13-формирование-импульса)
14. [Синхронизация](#14-синхронизация)


## PySDR.org 
 PySDR.org представляет собой вводный раздел учебника по программно-определяемому радио (SDR) и цифровой обработке сигналов (DSP) с использованием Python.
 
### 1 Введение
<details>
 <summary>Программно-Определяемое Радио</summary>
Радио, в котором задачи обработки сигналов, традиционно выполняемые аппаратными средствами, осуществляются с помощью программного обеспечения.
</details>
 
 <details><summary>Цифровая Обработка Сигналов</summary>
 Цифровая обработка сигналов, в данном случае радиочастотных сигналов.
 </details>

### 2 Частотная область
<details>
 <summary>Временная область и частотная область</summary>
Во-первых, почему нам нравится смотреть на сигналы в частотной области? Ну вот два примера сигналов, показанных во временной и частотной области.
   
   ![image](https://github.com/Depth-monster/proto_tcs/assets/122405130/d032937f-67bb-4d1f-b639-661a9cb1905f)

Как вы можете видеть, во временной области они оба в какой-то мере выглядят как шум, но в частотной области мы можем видеть различные характеристики. Всё находится во временной области в её естественной форме; когда мы семплируем сигналы, мы будем семплировать их во временной области, потому что вы не можете напрямую семплировать сигнал в частотной области. Но интересные вещи обычно происходят в частотной области.
</details>
 
 <details>
   <summary>Ряды Фурье...</summary>

### Ряды Фурье

Основы частотной области начинаются с понимания того, что любой сигнал может быть представлен как сумма синусоидальных волн. Когда мы разлагаем сигнал на его составные синусоидальные волны, мы называем это рядом Фурье. Вот пример сигнала, который состоит всего из двух синусоидальных волн:

![image](https://github.com/Depth-monster/proto_tcs/assets/122405130/f0de60ae-0019-468f-aa26-3012272cc5bd)


Здесь еще один пример; красная кривая внизу аппроксимирует пилообразную волну, суммируя до 10 синусоидальных волн. Мы видим, что это не идеальная реконструкция — потребовалось бы бесконечное количество синусоидальных волн, чтобы воспроизвести эту пилообразную волну из-за резких переходов:

![image](https://github.com/Depth-monster/proto_tcs/assets/122405130/a0a0a361-e77a-4ba4-b5c6-a8f65ee75427)


Некоторым сигналам требуется больше синусоидальных волн, чем другим, и некоторые требуют бесконечного количества, хотя их всегда можно аппроксимировать с ограниченным числом. Вот еще один пример сигнала, разложенного на ряд синусоидальных волн:

![image](https://github.com/Depth-monster/proto_tcs/assets/122405130/9eafe997-5646-4b38-bd2f-165648bfdebc)

 </details>

 <details>
 <summary>FFT in Python</summary>
    
![image](https://github.com/Depth-monster/proto_tcs/assets/122405130/c8416cba-236f-4327-8d7e-d326d5fd476e)

```python
import numpy as np
import matplotlib.pyplot as plt
```
Импортируем необходимые библиотеки: numpy для работы с массивами и математических операций, matplotlib.pyplot для визуализации результатов.

```python
def fft(x):
    N = len(x)
    if N == 1:
        return x
```
Определяем функцию fft, которая будет рекурсивно вычислять преобразование Фурье входного массива x. Если длина массива x равна 1, возвращаем x, так как это базовый случай рекурсии.

```python
    twiddle_factors = np.exp(-2j * np.pi * np.arange(N//2) / N)
    x_even = fft(x[::2])
    x_odd = fft(x[1::2])
```
Рассчитываем коэффициенты Виттакера (twiddle factors) для рекурсивного деления сигнала на четные и нечетные индексы.

```python
    return np.concatenate([x_even + twiddle_factors * x_odd,
                           x_even - twiddle_factors * x_odd])

```
Собираем результаты для четной и нечетной частей, умножая нечетную часть на коэффициенты Виттакера и объединяя результаты.

```python
# Simulate a tone + noise
sample_rate = 1e6
f_offset = 0.2e6 # 200 kHz offset from carrier
N = 1024
t = np.arange(N)/sample_rate
s = np.exp(2j*np.pi*f_offset*t)
```
Создаем тональный сигнал с заданной частотой смещения f_offset от несущей частоты.
```python
n = (np.random.randn(N) + 1j*np.random.randn(N))/np.sqrt(2) # unity complex noise
r = s + n # 0 dB SNR

```
Генерируем комплексный шум и добавляем его к нашему тональному сигналу s, получая результат с отношением сигнал/шум (SNR) 0 дБ
```python
# Perform fft, fftshift, convert to dB
X = fft(r)
X_shifted = np.roll(X, N//2) # equivalent to np.fft.fftshift
X_mag = 10*np.log10(np.abs(X_shifted)**2)

```
Выполняем FFT для смешанного сигнала, сдвигаем нулевую частоту в центр и переводим результат в децибелы.
```python
# Plot results
f = np.linspace(sample_rate/-2, sample_rate/2, N)/1e6 # plt in MHz
plt.plot(f, X_mag)
plt.plot(f[np.argmax(X_mag)], np.max(X_mag), 'rx') # show max
plt.grid()
plt.xlabel('Frequency [MHz]')
plt.ylabel('Magnitude [dB]')
plt.show() 

```
Строим график величины FFT в децибелах, отмечаем максимальное значение красным крестиком, и отображаем результаты. Частоты приведены в мегагерцах (MHz), а величина – в децибелах (dB).
</details>
 
 <details>
   <summary> </summary>
 
 </details>
 
### 3 Выборка IQ
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 
### 4 Цифровая модуляция
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

### 5 PlutoSDR в Python
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 
### 6 USRP в Python
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 
### 7 Шум и дБ
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 

### 8 Фильтры
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

### 9 Link budgets()
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 
### 10 Кодирование канала
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

### 11 IQ файлы и SigMF
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 
### 12 Многолучевое затухание
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 
### 13 Формирование импульса
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 
### 14 Синхронизация 
<details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>

 <details>
 <summary> </summary>

</details>
 
 <details>
   <summary> </summary>
 
 </details>
 

# Task_8 — проект на C#

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;
using System.Linq;
using System.Runtime.Remoting.Metadata.W3cXsd2001;
using System.Text;
using System.Threading.Tasks;
using System.Xml.Schema;

namespace Task_8
{
    class Program
    {
        static void Main()
        {
```

### Упаковка параметров в одно число

Задаются температура, давление, положение заслонки и флаг обледенения. Побитовые операции помещают эти значения в разные биты переменной `data`.

```csharp
            int temperature = -18; 
            int pressure = 120;    
            int damper = 65;      
            int ice = 1;           

            int data = (temperature + 40) & 127;
            data |= (pressure & 255) << 7;
            data |= (damper & 127) << 15;
            data |= (ice & 1) << 22;
```

### Вычисление и установка бита чётности

Цикл `for` проверяет первые 23 бита числа и считает количество единиц. Если единиц нечётное количество, устанавливается бит 23, чтобы общее количество единичных битов стало чётным.

```csharp
            int ones = 0;
            int copy = data;

            for (int i = 0; i < 23; i++)
            {
                ones += copy & 1;
                copy >>= 1;
            }

            if ((ones & 1) != 0)
                data |= 1 << 23;
```

### Извлечение и вывод параметров

Побитовые маски и сдвиги извлекают значения из `data`. Затем программа выводит параметры и проверяет чётность всех 24 битов.

```csharp
            int tempResult = (data & 127) - 40;
            int pressureResult = (data >> 7) & 255;
            int damperResult = (data >> 15) & 127;
            int iceResult = (data >> 22) & 1;

            Console.WriteLine("1. Упаковка данных");
            Console.WriteLine("Температура: " + tempResult);
            Console.WriteLine("Давление: " + pressureResult);
            Console.WriteLine("Заслонка: " + damperResult);
            Console.WriteLine("Обледенение: " + iceResult);

            ones = 0;
            copy = data;

            for (int i = 0; i < 24; i++)
            {
                ones += copy & 1;
                copy >>= 1;
            }

            Console.WriteLine("Чётность правильная: " + ((ones & 1) == 0));
```

### Создание и вывод матрицы

Создаётся двумерный массив температур. Метод `PrintMatrix()` выводит его до изменения, чтобы можно было сравнить результат после переворота.

```csharp
            int[,] matrix = {
                { -18, -17, -16, -19, -20 },
                { -15, -14, -13, -12, -11 },
                { -10, -9, -8, -7, -6 },
                { -5, -4, -3, -2, -1 },
                { -21, -22, -23, -24, -25 }
            };

            Console.WriteLine("\n2. Матрица до переворота");
            PrintMatrix(matrix);
```

### Горизонтальный переворот строк матрицы

Вложенные циклы проходят по строкам и половине столбцов. Каждый элемент меняется местами с симметричным элементом в той же строке, поэтому строки переворачиваются слева направо.

```csharp
            for (int i = 0; i < matrix.GetLength(0); i++)
            {
                for (int j = 0; j < matrix.GetLength(1) / 2; j++)
                {
                    int temp = matrix[i, j];
                    matrix[i, j] = matrix[i, matrix.GetLength(1) - 1 - j];
                    matrix[i, matrix.GetLength(1) - 1 - j] = temp;
                }
            }

            Console.WriteLine("Матрица после переворота");
            PrintMatrix(matrix);
```

### Подготовка данных и запуск размораживания

Создаются массивы температур полок и флагов обледенения. Метод `DefrostCycle()` меняет данные полок, на которых есть лёд, а параметр `out` возвращает количество размороженных полок.

```csharp
            int[] shelfTemps = { -12, -8, -15, -5, -10 };
            bool[] iceFlags = { true, false, true, true, false };
            int defrosted;

            DefrostCycle(ref shelfTemps, ref iceFlags, out defrosted, -5, -5, -5, -5, -5);

            Console.WriteLine("\n3. Размораживание");
            Console.WriteLine("Разморожено полок: " + defrosted);
            Console.WriteLine("Температуры полок:");

            for (int i = 0; i < shelfTemps.Length; i++)
                Console.Write(shelfTemps[i] + " ");

            Console.WriteLine();
```

### Моделирование роста льда

Метод `SimulateFrostBuildup()` получает начальную толщину льда, скорость роста и количество часов. Он рекурсивно прибавляет скорость роста, пока часы не закончатся.

```csharp
            double iceThickness = SimulateFrostBuildup(1, 0.5, 4);
            Console.WriteLine("\n4. Толщина льда через 4 часа: " + iceThickness + " мм");
```

### Расчёт мощности охлаждения

Сначала вычисляется базовая хладопроизводительность. Затем передаются температуры отсеков и целевая температура; второй метод возвращает требуемую мощность и индекс самого тёплого отсека.

```csharp
            int basePower = ComputeCoolingPower(1500, 25);
            int[] compartmentTemps = { -10, -5, -14, -3, -8 };
            int warmestCompartmentIndex;
            int requiredPower = ComputeCoolingPower(compartmentTemps, -18, out warmestCompartmentIndex);

            Console.WriteLine("\n5. Мощность охлаждения");
            Console.WriteLine("Базовая хладопроизводительность: " + basePower);
            Console.WriteLine("Требуемая мощность: " + requiredPower);
            Console.WriteLine("Индекс самого тёплого отсека: " + warmestCompartmentIndex);
        }
```

### Метод вывода матрицы

Метод получает двумерный массив и печатает его построчно. После каждого ряда выводится переход на новую строку.

```csharp
        static void PrintMatrix(int[,] matrix)
        {
            for (int i = 0; i < matrix.GetLength(0); i++)
            {
                for (int j = 0; j < matrix.GetLength(1); j++)
                    Console.Write(matrix[i, j] + "\t");

                Console.WriteLine();
            }
        }
```

### Метод размораживания полок

Метод перебирает полки и обрабатывает только те, у которых `iceFlags[i] == true`. Температура увеличивается на 5, при необходимости ограничивается безопасным пределом, флаг льда сбрасывается, а счётчик увеличивается.

```csharp
        static void DefrostCycle(ref int[] shelfTemps, ref bool[] iceFlags,
                                 out int defrostedShelves, params int[] safeLimits)
        {
            defrostedShelves = 0;

            for (int i = 0; i < shelfTemps.Length; i++)
            {
                if (iceFlags[i])
                {
                    shelfTemps[i] += 5;

                    if (i < safeLimits.Length && shelfTemps[i] < safeLimits[i])
                        shelfTemps[i] = safeLimits[i];

                    iceFlags[i] = false;
                    defrostedShelves++;
                }
            }
        }
```

### Рекурсивный метод роста льда

Если `hoursLeft` меньше или равен нулю, метод возвращает текущую толщину. Иначе прибавляет скорость роста и вызывает сам себя с количеством часов на единицу меньше.

```csharp
        static double SimulateFrostBuildup(double currentIceMm, double rate, int hoursLeft)
        {
            if (hoursLeft <= 0)
                return currentIceMm;

            return SimulateFrostBuildup(currentIceMm + rate, rate, hoursLeft - 1);
        }
```

### Расчёт базовой мощности

Метод рассчитывает мощность по формуле `compressorRpm / 10 - ambientTemp * 2`. Если результат отрицательный, он заменяется на ноль.

```csharp
        static int ComputeCoolingPower(int compressorRpm, int ambientTemp)
        {
            int power = compressorRpm / 10 - ambientTemp * 2;

            if (power < 0)
                power = 0;

            return power;
        }
```

### Расчёт мощности для нескольких отсеков

Метод суммирует разницу между температурой каждого отсека и целевой температурой для тех отсеков, которые теплее цели. Параллельно он определяет индекс самого тёплого отсека и возвращает сумму, умноженную на 10.

```csharp
        static int ComputeCoolingPower(int[] compartmentTemps, int targetTemp,
                                       out int warmestCompartmentIndex)
        {
            int power = 0;
            warmestCompartmentIndex = -1;

            if (compartmentTemps.Length == 0)
                return 0;

            warmestCompartmentIndex = 0;

            for (int i = 0; i < compartmentTemps.Length; i++)
            {
                if (compartmentTemps[i] > targetTemp)
                    power += compartmentTemps[i] - targetTemp;

                if (compartmentTemps[i] > compartmentTemps[warmestCompartmentIndex])
                    warmestCompartmentIndex = i;
            }

            return power * 10;
        }
    }
}
```

## Скриншот результата выполнения

Ниже приведён скриншот консоли с результатом запуска программы.

![Скриншот результата выполнения программы]<img width="1920" height="1080" alt="Happ Screenshot 2026 10 19 - 21 14 59 09" src="https://github.com/user-attachments/assets/d1ef6b37-ac35-4c67-9cd6-f282fb7145cf" />


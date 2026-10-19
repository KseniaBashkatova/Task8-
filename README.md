# Практическая работа №8: Рубежный практический контроль: Комплексный практикум по базовым конструкциям C#


### Вариант 6. Симулятор банкомата и кассет с банкнотами

```csharp

using System;

class Program
{
    static void Main()
    {
        Console.WriteLine("=== ЭТАП 1: Побитовая упаковка и маски ===");
        // Номинал: например, 5000 (индекс 5), остаток: 1500 купюр, статус: 2 (исправен), блокировка: 0 (не заблокирована)
        // Номиналы: 0->100, 1->200, 2->500, 3->1000, 4->2000, 5->5000
        byte nominalCode = 5;       // 4 бита
        ushort cassetteCount = 1500; // 12 бит
        byte dispenserStatus = 2;   // 2 бита
        byte isLocked = 0;          // 1 бит

        uint packedData = PackCassetteData(nominalCode, cassetteCount, dispenserStatus, isLocked);
        Console.WriteLine($"Упакованные данные (hex): 0x{packedData:X}");

        UnpackAndVerify(packedData);

        Console.WriteLine("\n=== ЭТАП 2: Сложная трансформация матрицы на месте ===");
        int[,] matrix = {
            { 100, 200, 500, 1000 },
            { 2000, 5000, 100, 200 },
            { 500, 1000, 2000, 5000 },
            { 10, 20, 50, 100 }
        };

        Console.WriteLine("Исходная матрица остатков кассет:");
        PrintMatrix(matrix);

        ShiftMatrixRowsDown(matrix);

        Console.WriteLine("Матрица после циклитического сдвига строк вниз на 1:");
        PrintMatrix(matrix);

        Console.WriteLine("\n=== ЭТАП 3: Конвейерная обработка массивов (ref, out, params) ===");
        int[] cassetteCounts = { 10, 5, 2, 20 }; // Количество купюр в кассетах
        int[] denominations = { 1000, 500, 200, 100 }; // Соответствующие номиналы
        int requestedSum = 3700;

        Console.WriteLine($"Запрошенная сумма: {requestedSum}");
        Console.WriteLine("Доступные купюры в кассетах: " + string.Join(", ", cassetteCounts));

        bool isSuccess;
        DispenseCash(ref cassetteCounts, requestedSum, out isSuccess, denominations);

        Console.WriteLine($"Результат выдачи: {(isSuccess ? "Успешно" : "Ошибка/Недостаточно средств")}");
        Console.WriteLine("Остаток купюр в кассетах после выдачи: " + string.Join(", ", cassetteCounts));

        Console.WriteLine("\n=== ЭТАП 4: Числовая рекурсия с условием выхода ===");
        int testSum = 1500;
        int baseNominal = 500;
        bool canSplit = CheckSumSplit(testSum, baseNominal);
        Console.WriteLine($"Сумма {testSum} делится на номинал {baseNominal} без остатка? {canSplit}");

        int testSum2 = 1300;
        bool canSplit2 = CheckSumSplit(testSum2, baseNominal);
        Console.WriteLine($"Сумма {testSum2} делится на номинал {baseNominal} без остатка? {canSplit2}");

        Console.WriteLine("\n=== ЭТАП 5: Перегрузка расчетных методов ===");
        int[] auditCounts = { 5, 10, 2, 1 };
        int[] auditDenoms = { 5000, 1000, 500, 100 };

        int totalMoney = AuditAtmBalance(auditCounts, auditDenoms);
        Console.WriteLine($"Суммарный остаток денег в банкомате: {totalMoney}");

        int minReserve = 3;
        int emptyMask;
        int totalMoneyWithAudit = AuditAtmBalance(auditCounts, auditDenoms, minReserve, out emptyMask);
        Console.WriteLine($"Суммарный остаток (с аудитом резерва): {totalMoneyWithAudit}");
        Console.WriteLine($"Маска кассет ниже минимального резерва ({minReserve}): 0x{emptyMask:X}");
    }

    // --- ЭТАП 1 ---
    static uint PackCassetteData(byte nominalCode, ushort cassetteCount, byte dispenserStatus, byte isLocked)
    {
        uint packed = 0;
        packed |= ((uint)(nominalCode & 0x0F)) << 28;
        packed |= ((uint)(cassetteCount & 0x0FFF)) << 16;
        packed |= ((uint)(dispenserStatus & 0x03)) << 14;
        packed |= ((uint)(isLocked & 0x01)) << 13;
        return packed;
    }

    static void UnpackAndVerify(uint packed)
    {
        byte nominalCode = (byte)((packed >> 28) & 0x0F);
        ushort cassetteCount = (ushort)((packed >> 16) & 0x0FFF);
        byte dispenserStatus = (byte)((packed >> 14) & 0x03);
        byte isLocked = (byte)((packed >> 13) & 0x01);

        Console.WriteLine($"Распаковано: Номинал-код={nominalCode}, Остаток={cassetteCount}, Статус={dispenserStatus}, Блокировка={isLocked}");

        // Проверка контрольного бита четности распакованных данных (сумма битов)
        int setBitsCount = 0;
        uint temp = packed;
        while (temp > 0)
        {
            setBitsCount += (int)(temp & 1);
            temp >>= 1;
        }

        // Без деления: проверяем младший бит счетчика setBitsCount через побитовую операцию & 1
        int parityBit = setBitsCount & 1;
        Console.WriteLine($"Контрольный бит четности packed-значения: {parityBit} ({(parityBit == 0 ? "Четное число единиц" : "Нечетное число единиц")})");
    }

    // --- ЭТАП 2 ---
    static void ShiftMatrixRowsDown(int[,] matrix)
    {
        int rows = matrix.GetLength(0);
        int cols = matrix.GetLength(1);
        if (rows <= 1) return;

        // Сохраняем последнюю строку во временный буфер-массив
        int[] buffer = new int[cols];
        for (int c = 0; c < cols; c++)
        {
            buffer[c] = matrix[rows - 1, c];
        }

        // Сдвигаем все строки сверху вниз на 1 позицию (in-place)
        for (int r = rows - 1; r > 0; r--)
        {
            for (int c = 0; c < cols; c++)
            {
                matrix[r, c] = matrix[r - 1, c];
            }
        }

        // Помещаем сохраненную последнюю строку на место первой
        for (int c = 0; c < cols; c++)
        {
            matrix[0, c] = buffer[c];
        }
    }

    static void PrintMatrix(int[,] matrix)
    {
        int rows = matrix.GetLength(0);
        int cols = matrix.GetLength(1);
        for (int r = 0; r < rows; r++)
        {
            for (int c = 0; c < cols; c++)
            {
                Console.Write($"{matrix[r, c],6} ");
            }
            Console.WriteLine();
        }
    }

    // --- ЭТАП 3 ---
    static void DispenseCash(ref int[] cassetteCounts, int requestedSum, out bool isSuccess, params int[] denominations)
    {
        isSuccess = false;
        if (requestedSum <= 0 || cassetteCounts.Length != denominations.Length)
        {
            return;
        }

        // Создаем копию счетчиков для симуляции жадного алгоритма, чтобы не изменять состояние при нехватке средств
        int[] tempCounts = new int[cassetteCounts.Length];
        for (int i = 0; i < cassetteCounts.Length; i++)
        {
            tempCounts[i] = cassetteCounts[i];
        }

        int remainingSum = requestedSum;

        // Жадный расчет выдачи
        for (int i = 0; i < denominations.Length; i++)
        {
            if (remainingSum <= 0) break;

            int nominal = denominations[i];
            int availableNotes = tempCounts[i];

            int neededNotes = remainingSum / nominal;
            int notesToTake = neededNotes < availableNotes ? neededNotes : availableNotes;

            tempCounts[i] -= notesToTake;
            remainingSum -= notesToTake * nominal;
        }

        // Если сумма полностью собрана
        if (remainingSum == 0)
        {
            isSuccess = true;
            cassetteCounts = tempCounts; // Переприсваиваем измененный массив через ref
        }
    }

    // --- ЭТАП 4 ---
    static bool CheckSumSplit(int sum, int stepNominal)
    {
        // Условие выхода
        if (sum == 0) return true;
        if (sum < 0 || stepNominal <= 0) return false;

        // Рекурсивный проход с уменьшением суммы на номинал
        return CheckSumSplit(sum - stepNominal, stepNominal);
    }

    // --- ЭТАП 5 ---
    static int AuditAtmBalance(int[] cassetteCounts, int[] denominations)
    {
        int total = 0;
        int length = cassetteCounts.Length < denominations.Length ? cassetteCounts.Length : denominations.Length;
        for (int i = 0; i < length; i++)
        {
            total += cassetteCounts[i] * denominations[i];
        }
        return total;
    }

    static int AuditAtmBalance(int[] cassetteCounts, int[] denominations, int minReserve, out int emptyCassettesMask)
    {
        int total = 0;
        emptyCassettesMask = 0;
        int length = cassetteCounts.Length < denominations.Length ? cassetteCounts.Length : denominations.Length;

        for (int i = 0; i < length; i++)
        {
            total += cassetteCounts[i] * denominations[i];

            // Если остаток в кассете строго ниже минимального резерва, устанавливаем бит в маске
            if (cassetteCounts[i] < minReserve)
            {
                emptyCassettesMask |= (1 << i);
            }
        }

        return total;
    }
}


```
`Результат выполнения:`
<picture>
  <img src="https://github.com/KseniaBashkatova/Task8-/blob/main/111.png?raw=true">
</picture>
```


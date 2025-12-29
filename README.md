# Лабораторная работа 5 – Файловый ввод-вывод

## Задача 1. Чтение и вывод содержимого текстового файла

**Постановка задачи:**  
Напишите программу, которая открывает текстовый файл (например, “input.txt”) для чтения, считывает его построчно с помощью функции fgets() и выводит каждую строку на стандартный вывод. Программа должна проверять, успешно ли открыт файл, и корректно закрывать его после чтения.

**Математическая модель:**  
Не требуется.

**Список идентификаторов:**

| Имя переменной | Тип данных         | Смысловое обозначение               |
|----------------|--------------------|-------------------------------------|
| fp             | FILE*              | Указатель на файловый поток         |
| buffer         | char[256]          | Буфер для хранения строки           |

**Код программы:**

```c
#include <stdio.h>
#include <stdlib.h>

void readAndPrintFile() {
    FILE *fp = fopen("input.txt", "r");
    if (fp == NULL) {
        perror("Ошибка открытия файла");
        exit(EXIT_FAILURE);
    }

    char buffer[256];
    while (fgets(buffer, sizeof(buffer), fp) != NULL) {
        printf("%s", buffer);
    }
    fclose(fp);
}

int main(void) {
    readAndPrintFile();
    return 0;
}
```

**Результаты выполненной работы:**  
(Скриншот вывода программы с содержимым файла input.txt)

---

## Задача 2. Запись пользовательского ввода в текстовый файл

**Постановка задачи:**  
Создайте программу, которая запрашивает у пользователя несколько строк текста, а затем записывает введённые данные в файл “output.txt”. Используйте режим записи “w”. После завершения записи файл закрывается, а программа выводит сообщение об успешном завершении.

**Математическая модель:**  
Не требуется.

**Список идентификаторов:**

| Имя переменной | Тип данных | Смысловое обозначение               |
|----------------|------------|-------------------------------------|
| fp             | FILE*      | Указатель на файловый поток         |
| line           | char[256]  | Строка, вводимая пользователем      |

**Код программы:**

```c
#include <stdio.h>
#include <stdlib.h>

void writeUserInputToFile() {
    FILE *fp = fopen("output.txt", "w");
    if (fp == NULL) {
        perror("Ошибка создания файла");
        exit(EXIT_FAILURE);
    }

    char line[256];
    printf("Введите строки текста (для завершения введите пустую строку):\n");
    while (1) {
        fgets(line, sizeof(line), stdin);
        if (line[0] == '\n') break;
        fputs(line, fp);
    }
    fclose(fp);
    printf("Данные успешно записаны в output.txt\n");
}

int main(void) {
    writeUserInputToFile();
    return 0;
}
```

**Результаты выполненной работы:**  
(Скриншот программы с вводом строк и сообщением об успешной записи)

---

## Задача 3. Копирование содержимого одного файла в другой

**Постановка задачи:**  
Напишите программу, которая копирует содержимое файла “source.txt” в новый файл “destination.txt”. Программа должна открывать исходный файл в режиме чтения, а целевой — в режиме записи. Содержимое копируется блоками (например, по 256 байт) с использованием функций fread() и fwrite().

**Математическая модель:**  
Не требуется.

**Список идентификаторов:**

| Имя переменной | Тип данных     | Смысловое обозначение               |
|----------------|----------------|-------------------------------------|
| src            | FILE*          | Исходный файл                       |
| dest           | FILE*          | Файл назначения                     |
| buffer         | char[256]      | Буфер для чтения/записи             |
| bytes          | size_t         | Количество прочитанных байт         |

**Код программы:**

```c
#include <stdio.h>
#include <stdlib.h>

void copyFile(const char *source, const char *destination) {
    FILE *src = fopen(source, "rb");
    if (src == NULL) {
        perror("Ошибка открытия исходного файла");
        exit(EXIT_FAILURE);
    }
    FILE *dest = fopen(destination, "wb");
    if (dest == NULL) {
        perror("Ошибка создания файла назначения");
        fclose(src);
        exit(EXIT_FAILURE);
    }

    char buffer[256];
    size_t bytes;
    while ((bytes = fread(buffer, 1, sizeof(buffer), src)) > 0) {
        fwrite(buffer, 1, bytes, dest);
    }
    fclose(src);
    fclose(dest);
    printf("Файл успешно скопирован из %s в %s\n", source, destination);
}

int main(void) {
    copyFile("source.txt", "destination.txt");
    return 0;
}
```

**Результаты выполненной работы:**  
(Скриншот с сообщением об успешном копировании)

---

## Задача 4. Подсчет строк, слов и символов в текстовом файле

**Постановка задачи:**  
Разработайте программу, которая открывает текстовый файл (например, “input.txt”) и подсчитывает: количество строк (по числу символов новой строки), количество слов (слова разделены пробелами и знаками препинания), количество символов (включая пробелы). После подсчета программа выводит результаты.

**Математическая модель:**  
Не требуется.

**Список идентификаторов:**

| Имя переменной | Тип данных | Смысловое обозначение               |
|----------------|------------|-------------------------------------|
| fp             | FILE*      | Указатель на файловый поток         |
| lines          | int        | Количество строк                    |
| words          | int        | Количество слов                     |
| chars          | int        | Количество символов                 |
| inWord         | int        | Флаг нахождения внутри слова (0/1)  |
| ch             | int        | Текущий символ из файла             |

**Код программы:**

```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>

void countFileStats(const char *filename) {
    FILE *fp = fopen(filename, "r");
    if (fp == NULL) {
        perror("Ошибка открытия файла");
        exit(EXIT_FAILURE);
    }

    int lines = 0, words = 0, chars = 0;
    int inWord = 0;
    int ch;

    while ((ch = fgetc(fp)) != EOF) {
        chars++;
        if (ch == '\n')
            lines++;
        if (isspace(ch))
            inWord = 0;
        else if (!inWord) {
            inWord = 1;
            words++;
        }
    }
    fclose(fp);
    printf("Строк: %d\nСлов: %d\nСимволов: %d\n", lines, words, chars);
}

int main(void) {
    countFileStats("input.txt");
    return 0;
}
```

**Результаты выполненной работы:**  
(Скриншот с результатами подсчёта)

---

## Задача 5. Запись и чтение структур в бинарном файле

**Постановка задачи:**  
Определите структуру (например, struct Student с полями name, age и grade). Создайте массив таких структур, затем запишите его в бинарный файл с помощью fwrite(). После этого откройте файл для чтения и восстановите массив с помощью fread(), после чего выведите данные на экран.

**Математическая модель:**  
Не требуется.

**Список идентификаторов:**

| Имя переменной    | Тип данных      | Смысловое обозначение               |
|-------------------|-----------------|-------------------------------------|
| studentsToWrite   | struct Student[]| Массив структур для записи          |
| studentsRead      | struct Student[]| Массив структур для чтения          |
| count             | int             | Количество записей                  |
| fp                | FILE*           | Указатель на файловый поток         |

**Код программы:**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Student {
    char name[50];
    int age;
    float grade;
};

void writeStudents(const char *filename, struct Student students[], int count) {
    FILE *fp = fopen(filename, "wb");
    if (fp == NULL) {
        perror("Ошибка создания бинарного файла");
        exit(EXIT_FAILURE);
    }
    fwrite(students, sizeof(struct Student), count, fp);
    fclose(fp);
}

void readStudents(const char *filename, struct Student students[], int count) {
    FILE *fp = fopen(filename, "rb");
    if (fp == NULL) {
        perror("Ошибка открытия бинарного файла");
        exit(EXIT_FAILURE);
    }
    fread(students, sizeof(struct Student), count, fp);
    fclose(fp);
}

int main(void) {
    struct Student studentsToWrite[2] = {
        {"Иванов", 20, 4.5f},
        {"Петров", 22, 3.8f}
    };
    writeStudents("students.bin", studentsToWrite, 2);

    struct Student studentsRead[2];
    readStudents("students.bin", studentsRead, 2);

    for (int i = 0; i < 2; i++) {
        printf("Студент: %s, возраст: %d, Оценка: %.2f\n",
               studentsRead[i].name, studentsRead[i].age, studentsRead[i].grade);
    }
    return 0;
}
```

**Результаты выполненной работы:**  
(Скриншот с выводом данных студентов)

---

## Задача 6. Произвольный доступ к файлу с использованием fseek()

**Постановка задачи:**  
Напишите программу, которая открывает бинарный файл с записями (например, структура из задачи 5). С помощью функции fseek() переместитесь к определённой записи (например, к записи с индексом 1), измените её данные и запишите изменения в файл. Затем прочитайте файл заново и выведите изменённую запись.

**Математическая модель:**  
Не требуется.

**Список идентификаторов:**

| Имя переменной | Тип данных      | Смысловое обозначение               |
|----------------|-----------------|-------------------------------------|
| students       | struct Student[]| Массив структур                     |
| newRecord      | struct Student  | Новая запись для обновления         |
| fp             | FILE*           | Указатель на файловый поток         |
| index          | int             | Индекс записи для изменения         |

**Код программы:**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Student {
    char name[50];
    int age;
    float grade;
};

void updateStudentRecord(const char *filename, int index, struct Student newData) {
    FILE *fp = fopen(filename, "rb+");
    if (fp == NULL) {
        perror("Ошибка открытия файла");
        exit(EXIT_FAILURE);
    }
    fseek(fp, index * sizeof(struct Student), SEEK_SET);
    fwrite(&newData, sizeof(struct Student), 1, fp);
    fclose(fp);
}

void printStudentRecord(const char *filename, int index) {
    FILE *fp = fopen(filename, "rb");
    if (fp == NULL) {
        perror("Ошибка открытия файла");
        exit(EXIT_FAILURE);
    }
    struct Student s;
    fseek(fp, index * sizeof(struct Student), SEEK_SET);
    fread(&s, sizeof(struct Student), 1, fp);
    fclose(fp);
    printf("Обновлённая запись: %s, %d, %.2f\n", s.name, s.age, s.grade);
}

int main(void) {
    struct Student students[2] = {
        {"Иванов", 20, 4.5f},
        {"Петров", 22, 3.8f}
    };

    FILE *fp = fopen("students.bin", "wb");
    if (fp == NULL) {
        perror("Ошибка создания файла");
        exit(EXIT_FAILURE);
    }
    fwrite(students, sizeof(struct Student), 2, fp);
    fclose(fp);

    struct Student newRecord = {"Петров", 23, 4.2f};
    updateStudentRecord("students.bin", 1, newRecord);

    printStudentRecord("students.bin", 1);
    return 0;
}
```

**Результаты выполненной работы:**  
(Скриншот с обновлённой записью)

---

## Задача 7. Использование временного файла (temporary file)

**Постановка задачи:**  
Разработайте программу, которая создает временный файл с помощью функции tmpfile(), записывает в него несколько строк (например, результаты промежуточных вычислений или лог), затем перемещается в начало файла с помощью fseek(), считывает содержимое и выводит его на экран. Временный файл автоматически удаляется после закрытия.

**Математическая модель:**  
Не требуется.

**Список идентификаторов:**

| Имя переменной | Тип данных | Смысловое обозначение               |
|----------------|------------|-------------------------------------|
| fp             | FILE*      | Указатель на временный файл         |
| buffer         | char[256]  | Буфер для чтения строк              |

**Код программы:**

```c
#include <stdio.h>
#include <stdlib.h>

void tempFileDemo() {
    FILE *fp = tmpfile();
    if (fp == NULL) {
        perror("Ошибка создания временного файла");
        exit(EXIT_FAILURE);
    }

    fprintf(fp, "Строка 1: Пример работы с tmpfile()\n");
    fprintf(fp, "Строка 2: Временный файл будет удален после закрытия\n");

    fseek(fp, 0, SEEK_SET);

    char buffer[256];
    while (fgets(buffer, sizeof(buffer), fp) != NULL) {
        printf("%s", buffer);
    }
    fclose(fp);
}

int main(void) {
    tempFileDemo();
    return 0;
}
```

**Результаты выполненной работы:**  
(Скриншот с выводом содержимого временного файла)

---

### Информация о студенте:
Иванова Елизавета, 1 курс, ПОО

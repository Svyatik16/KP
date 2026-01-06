# KP

Послный код:
#include <stdio.h>
#include <locale.h>
#include <string.h>
#include <stdlib.h>

enum Version {
    ALPHA,
    BETA,
    RELEASE,
    DEVELOPMENT,
    NONINFO
};

enum OperatingSystem {
    WINDOWS,
    LINUX,
    MACOS,
    OTHER
};

struct Date {
    int day;
    int month;
    int year;
};

struct Software {
    char name[100];
    double price;
    char info[200];
    enum Version version;
    struct Date releaseDate;
    int users;
    enum OperatingSystem os;
    double rating;
};

// Переменные для размера базы
#define MAX_SOFTWARE 1000
struct Software database[MAX_SOFTWARE];
int count = 0;

// Объявления функций
const char* versionToString(enum Version v);
const char* osToString(enum OperatingSystem os);
enum Version stringToVersion(const char* str);
enum OperatingSystem stringToOS(const char* str);
void saveToFile();
void loadFromFile();
void addRecord();
void viewAllRecords();
void searchByPrice();
void combinedSearch();
void sortData();
void showMenu();

int main() {
    setlocale(LC_ALL, "RUS");

    // Автоматически загружаем данные при запуске
    loadFromFile();

    int choice;

    do {
        showMenu();
        printf("Выберите действие: ");
        scanf("%d", &choice);

        switch (choice) {
        case 1:
            viewAllRecords();
            break;
        case 2:
            addRecord();
            break;
        case 3:
            saveToFile();
            break;
        case 4:
            combinedSearch();
            break;
        case 5:
            searchByPrice();
            break;
        case 6:
            sortData();
            break;
        case 7:
            loadFromFile();
            break;
        case 8:
            printf("Выход из программы...\n");
            break;
        default:
            printf("Неверный выбор! Попробуйте снова.\n");
        }

        printf("\n");

    } while (choice != 8);

    // Сохраняем данные при выходе
    saveToFile();

    return 0;
}

// Функция преобразования версии в строку
const char* versionToString(enum Version v) {
    switch (v) {
    case ALPHA: return "Alpha";
    case BETA: return "Beta";
    case RELEASE: return "Release";
    case DEVELOPMENT: return "Development";
    case NONINFO: return "NonInfo";
    default: return "Unknown";
    }
}

// Функция преобразования ОС в строку
const char* osToString(enum OperatingSystem os) {
    switch (os) {
    case WINDOWS: return "Windows";
    case LINUX: return "Linux";
    case MACOS: return "MacOS";
    case OTHER: return "Other";
    default: return "Unknown";
    }
}

// Функция преобразования строки в версию
enum Version stringToVersion(const char* str) {
    if (strcmp(str, "Alpha") == 0) return ALPHA;
    if (strcmp(str, "Beta") == 0) return BETA;
    if (strcmp(str, "Release") == 0) return RELEASE;
    if (strcmp(str, "Development") == 0) return DEVELOPMENT;
    return NONINFO;
}

// Функция преобразования строки в ОС
enum OperatingSystem stringToOS(const char* str) {
    if (strcmp(str, "Windows") == 0) return WINDOWS;
    if (strcmp(str, "Linux") == 0) return LINUX;
    if (strcmp(str, "MacOS") == 0) return MACOS;
    return OTHER;
}

// Функция сохранения данных в файл
void saveToFile() {
    FILE* file = fopen("database.txt", "w");

    if (file == NULL) {
        printf("Ошибка: не удалось создать/открыть файл для записи!\n");
        return;
    }

    // Записываем заголовок
    fprintf(file, "# База данных программного обеспечения\n");
    fprintf(file, "# Формат: Название|Цена|Информация|Версия|Дата(день.месяц.год)|Пользователи|ОС|Рейтинг\n\n");

    // Записываем все записи
    for (int i = 0; i < count; i++) {
        fprintf(file, "%s|%.2f|%s|%s|%d.%d.%d|%d|%s|%.1f\n",
            database[i].name,
            database[i].price,
            database[i].info,
            versionToString(database[i].version),
            database[i].releaseDate.day,
            database[i].releaseDate.month,
            database[i].releaseDate.year,
            database[i].users,
            osToString(database[i].os),
            database[i].rating);
    }

    fclose(file);

    printf("Данные успешно сохранены в файл 'database.txt'\n");
    printf("Сохранено записей: %d\n", count);
}

// Функция загрузки данных из файла
void loadFromFile() {
    FILE* file = fopen("database.txt", "r");

    if (file == NULL) {
        printf("Файл 'database.txt' не найден.\n");
        printf("Будет создан новый файл при сохранении.\n");
        return;
    }

    char line[500];
    count = 0;

    // Читаем файл построчно
    while (fgets(line, sizeof(line), file)) {
        // Пропускаем комментарии и пустые строки
        if (line[0] == '#' || line[0] == '\n') {
            continue;
        }

        struct Software soft;
        char versionStr[20];
        char dateStr[20];
        char osStr[20];

        // Строка с разделителем "|"
        if (sscanf(line, "%99[^|]|%lf|%199[^|]|%19[^|]|%19[^|]|%d|%19[^|]|%lf",
            soft.name,
            &soft.price,
            soft.info,
            versionStr,
            dateStr,
            &soft.users,
            osStr,
            &soft.rating) == 8) {

            // Преобразуем строки в перечисления
            soft.version = stringToVersion(versionStr);
            soft.os = stringToOS(osStr);

            // Дата
            sscanf(dateStr, "%d.%d.%d",
                &soft.releaseDate.day,
                &soft.releaseDate.month,
                &soft.releaseDate.year);

            // Добавляем в базу данных
            if (count < MAX_SOFTWARE) {
                database[count] = soft;
                count++;
            }
        }
    }

    fclose(file);
    printf("Загружено записей: %d\n", count);
}

// Функция добавления новой записи
void addRecord() {
    if (count >= MAX_SOFTWARE) {
        printf("Ошибка: достигнут лимит записей (%d)!\n", MAX_SOFTWARE);
        return;
    }

    printf("\n=== ДОБАВЛЕНИЕ НОВОЙ ЗАПИСИ ===\n");

    printf("Введите название программы: ");
    scanf(" %[^\n]", database[count].name);

    printf("Введите цену: ");
    scanf("%lf", &database[count].price);

    printf("Введите дополнительную информацию: ");
    scanf(" %[^\n]", database[count].info);

    printf("Выберите версию:\n");
    printf("0 - Alpha\n");
    printf("1 - Beta\n");
    printf("2 - Release\n");
    printf("3 - Development\n");
    printf("4 - NonInfo\n");
    printf("Ваш выбор: ");
    int ver;
    scanf("%d", &ver);
    database[count].version = (enum Version)ver;

    printf("Введите дату выпуска (день месяц год через пробел): ");
    scanf("%d %d %d", &database[count].releaseDate.day, &database[count].releaseDate.month, &database[count].releaseDate.year);

    printf("Введите количество пользователей: ");
    scanf("%d", &database[count].users);

    printf("Выберите операционную систему:\n");
    printf("0 - Windows\n");
    printf("1 - Linux\n");
    printf("2 - MacOS\n");
    printf("3 - Other\n");
    printf("Ваш выбор: ");
    int os;
    scanf("%d", &os);
    database[count].os = (enum OperatingSystem)os;

    printf("Введите рейтинг (от 0.0 до 5.0): ");
    scanf("%lf", &database[count].rating);

    // Проверка рейтинга
    if (database[count].rating < 0.0) database[count].rating = 0.0;
    if (database[count].rating > 5.0) database[count].rating = 5.0;

    count++;
    printf("Запись успешно добавлена!\n");
}

// Функция просмотра всех записей
void viewAllRecords() {
    if (count == 0) {
        printf("База данных пуста!\n");
        return;
    }

    printf("\n=== ВСЕ ЗАПИСИ В БАЗЕ ДАННЫХ (%d) ===\n", count);
    for (int i = 0; i < count; i++) {
        printf("\n[Запись %d]\n", i + 1);
        printf("  Название: %s\n", database[i].name);
        printf("  Цена: %.2f\n", database[i].price);
        printf("  Информация: %s\n", database[i].info);
        printf("  Версия: %s\n", versionToString(database[i].version));
        printf("  Дата выпуска: %d.%d.%d\n", database[i].releaseDate.day, database[i].releaseDate.month, database[i].releaseDate.year);
        printf("  Количество пользователей: %d\n", database[i].users);
        printf("  Операционная система: %s\n", osToString(database[i].os));
        printf("  Рейтинг: %.1f/5.0\n", database[i].rating);
        printf("----------------------------------------");
    }
    printf("\n");
}

// Функция поиска по цене (одиночный поиск)
void searchByPrice() {
    if (count == 0) {
        printf("База данных пуста! Нечего искать.\n");
        return;
    }

    printf("\n=== ПОИСК ПО ЦЕНЕ ===\n");

    int choice;
    printf("Выберите тип поиска:\n");
    printf("1 - По максимальной цене (дешевле чем...)\n");
    printf("2 - По минимальной цене (дороже чем...)\n");
    printf("3 - В диапазоне цен\n");
    printf("Ваш выбор: ");
    scanf("%d", &choice);

    int found = 0;

    switch (choice) {
    case 1: { // По максимальной цене
        double maxPrice;
        printf("Введите максимальную цену: ");
        scanf("%lf", &maxPrice);

        printf("\nПрограммы дешевле %.2f:\n", maxPrice);
        for (int i = 0; i < count; i++) {
            if (database[i].price <= maxPrice) {
                printf("[%d] %s - %.2f руб.\n", i + 1, database[i].name, database[i].price);
                found++;
            }
        }
        break;
    }
    case 2: { // По минимальной цене
        double minPrice;
        printf("Введите минимальную цену: ");
        scanf("%lf", &minPrice);

        printf("\nПрограммы дороже %.2f:\n", minPrice);
        for (int i = 0; i < count; i++) {
            if (database[i].price >= minPrice) {
                printf("[%d] %s - %.2f руб.\n", i + 1, database[i].name, database[i].price);
                found++;
            }
        }
        break;
    }
    case 3: { // В диапазоне цен
        double minPrice, maxPrice;
        printf("Введите минимальную цену: ");
        scanf("%lf", &minPrice);
        printf("Введите максимальную цену: ");
        scanf("%lf", &maxPrice);

        printf("\nПрограммы в диапазоне от %.2f до %.2f:\n", minPrice, maxPrice);
        for (int i = 0; i < count; i++) {
            if (database[i].price >= minPrice && database[i].price <= maxPrice) {
                printf("[%d] %s - %.2f руб.\n", i + 1, database[i].name, database[i].price);
                found++;
            }
        }
        break;
    }
    default:
        printf("Неверный выбор!\n");
        return;
    }

    if (found == 0) {
        printf("По вашему запросу ничего не найдено.\n");
    }
    else {
        printf("Найдено записей: %d\n", found);
    }
}

// Функция комбинированного поиска
void combinedSearch() {
    if (count == 0) {
        printf("База данных пуста! Нечего искать.\n");
        return;
    }

    printf("\n=== КОМБИНИРОВАННЫЙ ПОИСК (Версия + Рейтинг) ===\n");

    // Выбор версии
    int ver;
    printf("Выберите версию для поиска:\n");
    printf("0 - Alpha\n");
    printf("1 - Beta\n");
    printf("2 - Release\n");
    printf("3 - Development\n");
    printf("4 - NonInfo\n");
    printf("Ваш выбор: ");
    scanf("%d", &ver);

    // Выбор минимального рейтинга
    double minRating;
    printf("Введите минимальный рейтинг (от 0.0 до 5.0): ");
    scanf("%lf", &minRating);

    if (minRating < 0.0) minRating = 0.0;
    if (minRating > 5.0) minRating = 5.0;

    int found = 0;
    printf("\nПрограммы с версией '%s' и рейтингом не ниже %.1f:\n", versionToString((enum Version)ver), minRating);

    for (int i = 0; i < count; i++) {
        if (database[i].version == ver && database[i].rating >= minRating) {
            printf("[%d] %s\n", i + 1, database[i].name);
            printf("    Версия: %s\n", versionToString(database[i].version));
            printf("    Рейтинг: %.1f/5.0\n", database[i].rating);
            printf("    Цена: %.2f руб.\n", database[i].price);
            printf("    --------------------\n");
            found++;
        }
    }

    if (found == 0) {
        printf("По вашему запросу ничего не найдено.\n");
    }
    else {
        printf("Найдено записей: %d\n", found);
    }
}

// Функция сортировки данных
void sortData() {
    if (count == 0) {
        printf("База данных пуста! Нечего сортировать.\n");
        return;
    }

    printf("\n=== СОРТИРОВКА ДАННЫХ ===\n");
    printf("Выберите тип сортировки:\n");
    printf("1 - По рейтингу (убывание)\n");
    printf("2 - По цене (возрастание)\n");
    printf("3 - По рейтингу (убывание) и по цене (возрастание)\n");
    printf("Ваш выбор: ");

    int choice;
    scanf("%d", &choice);

    //Пузырьковая сортировка
    for (int i = 0; i < count - 1; i++) {
        for (int j = 0; j < count - i - 1; j++) {
            int swap = 0;

            switch (choice) {
            case 1: // По рейтингу (убывание)
                if (database[j].rating < database[j + 1].rating) swap = 1;
                break;

            case 2: // По цене (возрастание)
                if (database[j].price > database[j + 1].price) swap = 1;
                break;

            case 3: // По рейтингу (убывание) и по цене (возрастание)
                if (database[j].rating < database[j + 1].rating) {
                    swap = 1;
                }
                else if (database[j].rating == database[j + 1].rating) {
                    // Если рейтинги равны, сортируем по цене (возрастание)
                    if (database[j].price > database[j + 1].price) swap = 1;
                }
                break;

            default:
                printf("Неверный выбор!\n");
                return;
            }

            if (swap) {
                // Меняем местами записи
                struct Software temp = database[j];
                database[j] = database[j + 1];
                database[j + 1] = temp;
            }
        }
    }

    printf("Данные успешно отсортированы!\n");
    printf("Для просмотра результата выберите 'Просмотреть все записи' в меню.\n");
}

// меню
void showMenu() {
    printf("\n========== ГЛАВНОЕ МЕНЮ ==========\n");
    printf("1. Просмотреть все записи\n");
    printf("2. Добавить новую запись\n");
    printf("3. Сохранение данных в файл\n");
    printf("4. Комбинированный поиск (версия + рейтинг)\n");
    printf("5. Поиск по цене\n");
    printf("6. Сортировка данных\n");
    printf("7. Загрузка данных из файла\n");
    printf("8. Выход\n");
}

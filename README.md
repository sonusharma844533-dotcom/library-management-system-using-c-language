# library-management-system-using-c-language
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Book {
    int id;
    char name[100];
    char author[100];
    int quantity;
};

void addBook();
void displayBooks();
void searchBook();
void issueBook();
void returnBook();
void deleteBook();

int main() {
    int choice;

    while (1) {
        printf("\n========== LIBRARY MANAGEMENT SYSTEM ==========\n");
        printf("1. Add New Book\n");
        printf("2. Display All Books\n");
        printf("3. Search Book\n");
        printf("4. Issue Book\n");
        printf("5. Return Book\n");
        printf("6. Delete Book\n");
        printf("7. Exit\n");

        printf("\nEnter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                addBook();
                break;

            case 2:
                displayBooks();
                break;

            case 3:
                searchBook();
                break;

            case 4:
                issueBook();
                break;

            case 5:
                returnBook();
                break;

            case 6:
                deleteBook();
                break;

            case 7:
                printf("\nThank you for using Library Management System!\n");
                exit(0);

            default:
                printf("\nInvalid Choice! Try Again.\n");
        }
    }

    return 0;
}

void addBook() {
    FILE *fp;
    struct Book b, temp;

    fp = fopen("library.dat", "ab+");

    if (fp == NULL) {
        printf("File opening error!\n");
        return;
    }

    printf("\nEnter Book ID: ");
    scanf("%d", &b.id);

    rewind(fp);
    while (fread(&temp, sizeof(temp), 1, fp) == 1) {
        if (temp.id == b.id) {
            printf("Book ID already exists!\n");
            fclose(fp);
            return;
        }
    }

    printf("Enter Book Name: ");
    scanf(" %99[^\n]", b.name);

    printf("Enter Author Name: ");
    scanf(" %99[^\n]", b.author);

    printf("Enter Quantity: ");
    scanf("%d", &b.quantity);

    if (b.quantity < 0) {
        printf("Invalid quantity!\n");
        fclose(fp);
        return;
    }

    fseek(fp, 0, SEEK_END);
    fwrite(&b, sizeof(b), 1, fp);

    printf("\nBook Added Successfully!\n");

    fclose(fp);
}

void displayBooks() {
    FILE *fp;
    struct Book b;
    int count = 0;

    fp = fopen("library.dat", "rb");

    if (fp == NULL) {
        printf("\nNo Books Available!\n");
        return;
    }

    printf("\n----------- BOOK RECORDS -----------\n");

    while (fread(&b, sizeof(b), 1, fp) == 1) {
        printf("\nBook ID: %d", b.id);
        printf("\nBook Name: %s", b.name);
        printf("\nAuthor: %s", b.author);
        printf("\nAvailable Quantity: %d\n", b.quantity);
        count++;
    }

    if (count == 0)
        printf("\nNo Books Found!\n");

    fclose(fp);
}

void searchBook() {
    FILE *fp;
    struct Book b;
    int id, found = 0;

    fp = fopen("library.dat", "rb");

    if (fp == NULL) {
        printf("\nNo Records Found!\n");
        return;
    }

    printf("\nEnter Book ID to Search: ");
    scanf("%d", &id);

    while (fread(&b, sizeof(b), 1, fp) == 1) {
        if (b.id == id) {
            printf("\nBook Found!");
            printf("\nID: %d", b.id);
            printf("\nName: %s", b.name);
            printf("\nAuthor: %s", b.author);
            printf("\nQuantity: %d\n", b.quantity);
            found = 1;
            break;
        }
    }

    if (!found)
        printf("\nBook Not Found!\n");

    fclose(fp);
}

void issueBook() {
    FILE *fp;
    struct Book b;
    int id, found = 0;

    fp = fopen("library.dat", "rb+");

    if (fp == NULL) {
        printf("\nNo Records Found!\n");
        return;
    }

    printf("\nEnter Book ID to Issue: ");
    scanf("%d", &id);

    while (fread(&b, sizeof(b), 1, fp) == 1) {
        if (b.id == id) {
            found = 1;

            if (b.quantity > 0) {
                b.quantity--;

                fseek(fp, -(long)sizeof(b), SEEK_CUR);
                fwrite(&b, sizeof(b), 1, fp);

                printf("\nBook Issued Successfully!\n");
            } else {
                printf("\nBook is Not Available!\n");
            }
            break;
        }
    }

    if (!found)
        printf("\nBook Not Found!\n");

    fclose(fp);
}

void returnBook() {
    FILE *fp;
    struct Book b;
    int id, found = 0;

    fp = fopen("library.dat", "rb+");

    if (fp == NULL) {
        printf("\nNo Records Found!\n");
        return;
    }

    printf("\nEnter Book ID to Return: ");
    scanf("%d", &id);

    while (fread(&b, sizeof(b), 1, fp) == 1) {
        if (b.id == id) {
            found = 1;
            b.quantity++;

            fseek(fp, -(long)sizeof(b), SEEK_CUR);
            fwrite(&b, sizeof(b), 1, fp);

            printf("\nBook Returned Successfully!\n");
            break;
        }
    }

    if (!found)
        printf("\nBook Not Found!\n");

    fclose(fp);
}

void deleteBook() {
    FILE *fp, *temp;
    struct Book b;
    int id, found = 0;

    fp = fopen("library.dat", "rb");

    if (fp == NULL) {
        printf("\nNo Records Found!\n");
        return;
    }

    temp = fopen("temp.dat", "wb");

    if (temp == NULL) {
        printf("\nUnable to create temporary file!\n");
        fclose(fp);
        return;
    }

    printf("\nEnter Book ID to Delete: ");
    scanf("%d", &id);

    while (fread(&b, sizeof(b), 1, fp) == 1) {
        if (b.id == id) {
            found = 1;
        } else {
            fwrite(&b, sizeof(b), 1, temp);
        }
    }

    fclose(fp);
    fclose(temp);

    if (found) {
        remove("library.dat");
        rename("temp.dat", "library.dat");
        printf("\nBook Deleted Successfully!\n");
    } else {
        remove("temp.dat");
        printf("\nBook Not Found!\n");
    }
}

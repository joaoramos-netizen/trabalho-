#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define ARQ "livros.bin"

// ---------------------------
// ESTRUTURA DE DADOS
// ---------------------------
typedef struct {
    char titulo[50];
    char autor[50];
    int ano;
} Livro;

// ---------------------------
// PROTÓTIPOS
// ---------------------------
void limpaBuffer();
void ler_string(char *str, int tam);
int tamanho(FILE *fp);
void cadastrar(FILE *fp);
void consultar(FILE *fp);

// ---------------------------
// limpaBuffer()
// ---------------------------
void limpaBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

// ---------------------------
// Leitura segura de string
// ---------------------------
void ler_string(char *str, int tam) {
    fgets(str, tam, stdin);
    str[strcspn(str, "\n")] = '\0'; // remove \n
}

// ---------------------------
// tamanho()
// Retorna número total de registros
// ---------------------------
int tamanho(FILE *fp) {
    fseek(fp, 0, SEEK_END);
    long bytes = ftell(fp);
    return bytes / sizeof(Livro);
}

// ---------------------------
// cadastrar()
// ---------------------------
void cadastrar(FILE *fp) {
    Livro l;

    printf("\n--- Cadastro de Livro ---\n");

    printf("Título: ");
    ler_string(l.titulo, 50);

    printf("Autor: ");
    ler_string(l.autor, 50);

    printf("Ano: ");
    scanf("%d", &l.ano);
    limpaBuffer();

    fseek(fp, 0, SEEK_END);
    fwrite(&l, sizeof(Livro), 1, fp);

    printf("Livro cadastrado com sucesso!\n");
}

// ---------------------------
// consultar()
// ---------------------------
void consultar(FILE *fp) {
    int indice;
    Livro l;

    printf("\nInforme a posição do registro: ");
    scanf("%d", &indice);
    limpaBuffer();

    if (indice < 0 || indice >= tamanho(fp)) {
        printf("Registro inexistente.\n");
        return;
    }

    fseek(fp, indice * sizeof(Livro), SEEK_SET);
    fread(&l, sizeof(Livro), 1, fp);

    printf("\n--- Dados do Livro ---\n");
    printf("Título: %s\n", l.titulo);
    printf("Autor: %s\n", l.autor);
    printf("Ano: %d\n", l.ano);
}

// ---------------------------
// main()
// ---------------------------
int main() {
    FILE *fp = fopen(ARQ, "r+b");

    if (!fp) {
        fp = fopen(ARQ, "w+b");
        if (!fp) {
            printf("Erro ao abrir/criar o arquivo.\n");
            return 1;
        }
    }

    int opc;

    do {
        printf("\n======= MENU =======\n");
        printf("1 - Cadastrar Livro\n");
        printf("2 - Consultar Livro\n");
        printf("3 - Total de Registros\n");
        printf("0 - Sair\n");
        printf("Escolha: ");
        scanf("%d", &opc);
        limpaBuffer();

        switch(opc) {
            case 1:
                cadastrar(fp);
                break;
            case 2:
                consultar(fp);
                break;
            case 3:
                printf("Total de livros: %d\n", tamanho(fp));
                break;
            case 0:
                printf("Encerrando...\n");
                break;
            default:
                printf("Opção inválida.\n");
        }

    } while (opc != 0);

    fclose(fp);
    return 0;
}

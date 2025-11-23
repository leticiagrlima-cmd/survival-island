# survival-island
Mini game em C — simulação de sobrevivência para o trabalho da disciplina.

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

#define MAX_INVENTORY 50
#define NAME_LEN 64
#define LINE_LEN 128

typedef enum { FOOD = 0, WEAPON = 1, TOOL = 2 } ItemType;
const char *ItemTypeNames[] = { "Food", "Weapon", "Tool" };

typedef struct {
    int id;
    char name[NAME_LEN];
    ItemType type;
    int priority;   // 1..10
    int quantity;
} Item;

/* Inventory as sequential array */
typedef struct {
    Item items[MAX_INVENTORY];
    int size;
    int next_id;
} Inventory;

void init_inventory(Inventory *inv) {
    inv->size = 0;
    inv->next_id = 1;
}

int add_item_inv(Inventory *inv, const char *name, ItemType type, int priority, int quantity) {
    if (inv->size >= MAX_INVENTORY) return 0;
    Item *it = &inv->items[inv->size++];
    it->id = inv->next_id++;
    strncpy(it->name, name, NAME_LEN-1);
    it->name[NAME_LEN-1] = '\0';
    it->type = type;
    it->priority = priority;
    it->quantity = quantity > 0 ? quantity : 1;
    return 1;
}

int remove_item_by_id(Inventory *inv, int id) {
    for (int i = 0; i < inv->size; ++i) {
        if (inv->items[i].id == id) {
            for (int j = i; j < inv->size - 1; ++j) inv->items[j] = inv->items[j + 1];
            inv->size--;
            return 1;
        }
    }
    return 0;
}

void print_item(const Item *it) {
    printf("[ID:%d] %-22s Type:%-6s Pri:%2d Qty:%2d\n",
           it->id, it->name, ItemTypeNames[it->type], it->priority, it->quantity);
}

void print_inventory(const Inventory *inv) {
    if (inv->size == 0) {
        printf("Inventário vazio.\n");
        return;
    }
    printf("\n--- Inventário (total: %d) ---\n", inv->size);
    for (int i = 0; i < inv->size; ++i) print_item(&inv->items[i]);
    printf("-------------------------------\n\n");
}

/* --- Linked list for comparison --- */
typedef struct Node {
    Item item;
    struct Node *next;
} Node;

Node *list_insert_end(Node *head, Item it) {
    Node *n = malloc(sizeof(Node));
    if (!n) return head;
    n->item = it;
    n->next = NULL;
    if (!head) return n;
    Node *cur = head;
    while (cur->next) cur = cur->next;
    cur->next = n;
    return head;
}

void list_print(Node *head) {
    if (!head) { printf("Lista encadeada vazia.\n"); return; }
    printf("\n--- Lista Encadeada ---\n");
    Node *cur = head;
    while (cur) {
        print_item(&cur->item);
        cur = cur->next;
    }
    printf("-----------------------\n\n");
}

void list_free(Node *head) {
    Node *cur = head;
    while (cur) {
        Node *tmp = cur;
        cur = cur->next;
        free(tmp);
    }
}

Node *inv_to_list(const Inventory *inv) {
    Node *head = NULL;
    for (int i = 0; i < inv->size; ++i) head = list_insert_end(head, inv->items[i]);
    return head;
}

void list_to_inv(Node *head, Inventory *inv) {
    init_inventory(inv);
    Node *cur = head;
    while (cur && inv->size < MAX_INVENTORY) {
        inv->items[inv->size++] = cur->item;
        cur = cur->next;
    }
}

/* --- Utility (case-insensitive compare) --- */
static int ci_strcmp(const char *a, const char *b) {
    for (; *a && *b; ++a, ++b) {
        int ca = tolower((unsigned char)*a);
        int cb = tolower((unsigned char)*b);
        if (ca != cb) return ca - cb;
    }
    return tolower((unsigned char)*a) - tolower((unsigned char)*b);
}

/* --- Swap --- */
void swap_items(Item *a, Item *b) { Item tmp = *a; *a = *b; *b = tmp; }

/* --- Selection Sort variants --- */
void selection_sort_by_name(Inventory *inv) {
    for (int i = 0; i < inv->size - 1; ++i) {
        int min = i;
        for (int j = i + 1; j < inv->size; ++j)
            if (ci_strcmp(inv->items[j].name, inv->items[min].name) < 0) min = j;
        if (min != i) swap_items(&inv->items[i], &inv->items[min]);
    }
}

void selection_sort_by_type_priority(Inventory *inv) {
    for (int i = 0; i < inv->size - 1; ++i) {
        int sel = i;
        for (int j = i + 1; j < inv->size; ++j) {
            if (inv->items[j].type < inv->items[sel].type) sel = j;
            else if (inv->items[j].type == inv->items[sel].type &&
                     inv->items[j].priority > inv->items[sel].priority)
                sel = j;
        }
        if (sel != i) swap_items(&inv->items[i], &inv->items[sel]);
    }
}

void selection_sort_by_priority(Inventory *inv) {
    for (int i = 0; i < inv->size - 1; ++i) {
        int max = i;
        for (int j = i + 1; j < inv->size; ++j)
            if (inv->items[j].priority > inv->items[max].priority) max = j;
        if (max != i) swap_items(&inv->items[i], &inv->items[max]);
    }
}

/* --- Binary search: works when sorted by name (case-insensitive) --- */
int binary_search_by_name(const Inventory *inv, const char *name) {
    int lo = 0, hi = inv->size - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int cmp = ci_strcmp(inv->items[mid].name, name);
        if (cmp == 0) return mid;
        if (cmp < 0) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

/* --- Exploration: collects items based on choice --- */
void explore_and_find(Inventory *inv) {
    printf("Você desembarcou na ilha. Há duas trilhas: esquerda (e) e direita (d).\n");
    cha

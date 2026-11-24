#include "DS.h"

#define RED 1
#define BLACK 0

static int isRed(Elm *p)
{
    return p != NULL && p->color == RED;
}

static Elm *rotateLeft(Elm *h)
{
    Elm *x = h->R;

    h->R = x->L;
    x->L = h;

    x->color = h->color;
    h->color = RED;

    return x;
}

static Elm *rotateRight(Elm *h)
{
    Elm *x = h->L;

    h->L = x->R;
    x->R = h;

    x->color = h->color;
    h->color = RED;

    return x;
}

static void flipColors(Elm *h)
{
    h->color = !h->color;

    if (h->L != NULL)
        h->L->color = !h->L->color;

    if (h->R != NULL)
        h->R->color = !h->R->color;
}

int less(const Student *a, const Student *b)
{
    return strcmp(a->id, b->id) < 0;
}

static Elm *insertNode(Elm *h, const Student *px)
{
    if (h == NULL) {
        Elm *q = malloc(sizeof(Elm));

        if (q == NULL)
            return NULL;

        q->x = *px;
        q->color = RED;
        q->L = NULL;
        q->R = NULL;

        return q;
    }

    if (less(px, &h->x)) {
        Elm *q = insertNode(h->L, px);

        if (q != NULL)
            h->L = q;
    }
    else if (less(&h->x, px)) {
        Elm *q = insertNode(h->R, px);

        if (q != NULL)
            h->R = q;
    }
    else {
        h->x = *px;
    }

    if (isRed(h->R) && !isRed(h->L))
        h = rotateLeft(h);

    if (isRed(h->L) && isRed(h->L->L))
        h = rotateRight(h);

    if (isRed(h->L) && isRed(h->R))
        flipColors(h);

    return h;
}

void rb_put(RBT *ptree, const Student *px)
{
    if (ptree == NULL || px == NULL)
        return;

    ptree->root = insertNode(ptree->root, px);

    if (ptree->root != NULL)
        ptree->root->color = BLACK;
}

Elm *rb_get(const RBT *ptree, const char id[])
{
    Elm *p;

    if (ptree == NULL || id == NULL)
        return NULL;

    p = ptree->root;

    while (p != NULL) {
        int cmp = strcmp(id, p->x.id);

        if (cmp < 0)
            p = p->L;
        else if (cmp > 0)
            p = p->R;
        else
            return p;
    }

    return NULL;
}

static Elm *moveRedLeft(Elm *h)
{
    flipColors(h);

    if (h->R != NULL && isRed(h->R->L)) {
        h->R = rotateRight(h->R);
        h = rotateLeft(h);
        flipColors(h);
    }

    return h;
}

static Elm *moveRedRight(Elm *h)
{
    flipColors(h);

    if (h->L != NULL && isRed(h->L->L)) {
        h = rotateRight(h);
        flipColors(h);
    }

    return h;
}

static Elm *balance(Elm *h)
{
    if (isRed(h->R))
        h = rotateLeft(h);

    if (isRed(h->L) && isRed(h->L->L))
        h = rotateRight(h);

    if (isRed(h->L) && isRed(h->R))
        flipColors(h);

    return h;
}

static Elm *minNode(Elm *h)
{
    while (h->L != NULL)
        h = h->L;

    return h;
}

static Elm *deleteMin(Elm *h)
{
    if (h->L == NULL) {
        free(h);
        return NULL;
    }

    if (!isRed(h->L) && !isRed(h->L->L))
        h = moveRedLeft(h);

    h->L = deleteMin(h->L);

    return balance(h);
}

static Elm *deleteNode(Elm *h, const char id[])
{
    if (strcmp(id, h->x.id) < 0) {
        if (h->L != NULL) {
            if (!isRed(h->L) && !isRed(h->L->L))
                h = moveRedLeft(h);

            h->L = deleteNode(h->L, id);
        }
    }
    else {
        if (isRed(h->L))
            h = rotateRight(h);

        if (strcmp(id, h->x.id) == 0 && h->R == NULL) {
            free(h);
            return NULL;
        }

        if (h->R != NULL) {
            if (!isRed(h->R) && !isRed(h->R->L))
                h = moveRedRight(h);

            if (strcmp(id, h->x.id) == 0) {
                Elm *m = minNode(h->R);

                h->x = m->x;
                h->R = deleteMin(h->R);
            }
            else {
                h->R = deleteNode(h->R, id);
            }
        }
    }

    return balance(h);
}

void rb_del(RBT *ptree, const char id[])
{
    if (ptree == NULL || ptree->root == NULL || id == NULL)
        return;

    if (rb_get(ptree, id) == NULL)
        return;

    if (!isRed(ptree->root->L) &&
        !isRed(ptree->root->R))
        ptree->root->color = RED;

    ptree->root = deleteNode(ptree->root, id);

    if (ptree->root != NULL)
        ptree->root->color = BLACK;
}

void print(Elm *p)
{
    if (p)
        printf("%s %s %d %.1f\n",
               p->x.name,
               p->x.id,
               p->x.age,
               p->x.gpa);
    else
        printf("None\n");
}
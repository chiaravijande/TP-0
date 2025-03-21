#include <stdio.h>
#include <stdlib.h>
#include "archivo.h"

struct Archivo {
    FILE *fp;
    char *buffer;
    size_t buffer_size;
    int lineas_leidas;
};

Archivo *archivo_abrir(const char *nombre) {
    Archivo *archivo = malloc(sizeof(Archivo));
    if (!archivo) return NULL;

    archivo->fp = fopen(nombre, "r");
    if (!archivo->fp) {
        free(archivo);
        return NULL;
    }

    archivo->buffer = NULL;
    archivo->buffer_size = 0;
    archivo->lineas_leidas = 0;

    return archivo;
}

const char *archivo_leer_linea(Archivo *archivo) {
    if (!archivo || !archivo->fp) return NULL;

    size_t capacidad = 10;  // Tamaño inicial del buffer
    size_t longitud = 0;  
    char c;

    free(archivo->buffer);
    archivo->buffer = malloc(capacidad);
    if (!archivo->buffer) return NULL;

    while ((c = fgetc(archivo->fp)) != EOF) {
        if (longitud + 1 >= capacidad) { 
            capacidad *= 2;  // Duplicamos el tamaño
            char *nuevo_buffer = realloc(archivo->buffer, capacidad);
            if (!nuevo_buffer) {
                free(archivo->buffer);
                archivo->buffer = NULL;
                return NULL;
            }
            archivo->buffer = nuevo_buffer;
        }

        archivo->buffer[longitud++] = c;
        if (c == '\n') break;
    }

    if (longitud == 0) {
        free(archivo->buffer);
        archivo->buffer = NULL;
        return NULL;
    }

    archivo->buffer[longitud] = '\0';
    archivo->lineas_leidas++;
    return archivo->buffer;
}

int archivo_hay_mas_lineas(Archivo *archivo) {
    if (!archivo || !archivo->fp) return 0;
    return !feof(archivo->fp);
}

int archivo_lineas_leidas(Archivo *archivo) {
    return archivo ? archivo->lineas_leidas : 0;
}

void archivo_cerrar(Archivo *archivo) {
    if (!archivo) return;
    
    if (archivo->fp) fclose(archivo->fp);
    free(archivo->buffer);
    free(archivo);
}

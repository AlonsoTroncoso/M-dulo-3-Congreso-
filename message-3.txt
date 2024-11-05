#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define MAX_CONGRESISTAS 200
#define MAX_SENADORES 50
#define MAX_DIPUTADOS 100
#define MAX_COMISIONES 10

struct congreso;
struct congresista;
struct nodoCongresista;
struct nodoProyectoLey;
struct proyectoLey;
struct nodoComision;
struct comision;
struct nodoArticulo;
struct articulo;
struct nodoVotacion;
struct votacion;

struct congresista {
    char *nombre;
    char *rut;
    char *ocupacion;
    char *especializacion;
};

struct nodoCongresista {
    struct congresista *datos;
    struct nodoCongresista *sig;
};

struct congreso {
    struct congresista **senadores;
    struct congresista **diputados;
    struct nodoCongresista *congresistasMixtos;
    struct nodoComision *comisionesMixtas;
    struct comision **comisiones;
    struct nodoProyectoLey *raiz;
};

struct nodoProyectoLey {
    struct proyectoLey *datos;
    struct nodoProyectoLey *izq, *der;
};

struct proyectoLey {
    char *nombre;
    char *tipo;
    int idProyecto;
    int urgencia;
    struct nodoArticulo *articulo;
    struct nodoVotacion *votacion;
    struct comision *comision;
    int fase;
};

struct nodoComision {
    struct comision *datos;
    struct nodoComision *sig;
};

struct comision {
    struct nodoCongresista *headIntegrantes;
    char *nombre;
    char *tipo;
    char *descripcion;
};


struct nodoArticulo {
    struct articulo *datos;
    struct nodoArticulo *sig, *ant;
};

struct articulo {
    char *nombre;
    int seccion;
    char *texto;
    char *cambio;
    struct votacion *voto;
};


struct nodoVotacion {
    struct votacion *datos;
    struct nodoVotacion *sig;
};

struct votacion {
    struct nodoCongresista *favor;
    struct nodoCongresista *contra;
    int fase;
};

// Function prototype
void limpiarBuffer();
void convertirMinusculas(char *cadena);
int leerEnteroConLimite(char *mensaje, int min, int max);
struct congresista *buscarCongresistaPorRUT(struct congreso *congreso, char *rut);
char leerOpcion();
struct congreso *inicializarCongreso();
void liberarCongreso(struct congreso *congreso);
struct nodoProyectoLey *crearNodoProyectoLey(struct proyectoLey *datos);
void agregarNodoProyectoLey(struct congreso *congreso);
struct proyectoLey *buscarProyectoLeyPorID(struct nodoProyectoLey *raiz, int id);
struct nodoProyectoLey *minValorNodo(struct nodoProyectoLey *nodo);
struct nodoProyectoLey *borrarNodoProyectoLey(struct congreso *congreso, struct nodoProyectoLey *raiz, int id);
void borrarProyectoLey(struct congreso *congreso, int id);
void mostrarCongresistasVotacion(struct nodoCongresista *lista, const char *categoria);
void buscarYMostrarProyectoLey(struct congreso *congreso, int id);
void limpiarBufferEntrada();
void agregarCongresistaAVotacion(struct votacion *votacion, struct congreso *congreso) ;
void agregarVotacion(struct congreso *congreso, int idProyecto);
void imprimirProyectoLey(struct proyectoLey *proyecto);
void recorrerYImprimirProyectos(struct nodoProyectoLey *nodo);
void imprimirProyectosLey(struct congreso *congreso);
void agregarProyecto(struct proyectoLey *proyecto, struct proyectoLey ***proyectosArray, int *proyectosCount, int *proyectosCapacidad);
void recorrerArbolEnOrden(struct nodoProyectoLey *nodo, struct proyectoLey ***proyectosArray, int *proyectosCount, int *proyectosCapacidad);
int compararPorUrgencia(const void *a, const void *b);
void mostrarProyectosOrdenDeUrgencia(struct congreso *congreso);
void modificarProyectoLey(struct congreso *congreso, int idProyecto);
struct nodoCongresista *crearNodoCongresista(struct nodoCongresista *head, struct congresista *datos);
int comprobarCongresistaEnComision(struct nodoCongresista *head, char *rutBuscado);
struct congresista* comprobarCongresistaEnCongreso(struct congreso *congreso, char *rutBuscado);
struct congresista *crearCongresista(struct congreso *congreso);
void agregarCongresistaEnCongreso(struct congreso *congreso);
void agregarCongresistaEnComision(struct comision *comision, char *rut, struct congresista *congresista);
void eliminarCongresistaDeComision(struct comision *comision, char *rutQuitado);
void eliminarCongresistaDeCongreso(struct congreso *congreso, char *rutQuitado);
void modificarCongresista(struct congreso *congreso, char *rutBuscado);
void mostrarCongresista(struct congreso *congreso, char *rutBuscado);
void listarCongresistas(struct congreso *congreso);
struct nodoComision* crearNodoComision(struct comision *datos);
struct comision *buscarComision(struct congreso *congreso, char *nombre);
struct comision *crearComision(struct congreso *congreso);
void agregarComision(struct congreso *congreso);
void eliminarComision(struct congreso *congreso, char *nombre);
void modificarComision(struct congreso *congreso, char *nombre);
void mostrarComisionPorNombre(struct congreso *congreso, char *nombre);
void listarComisiones(struct congreso *congreso);
void copiarCambioATexto(struct nodoArticulo *articulos);
struct nodoArticulo *crearNodoArticulo(struct articulo *datos);
int comprobarArticulo(struct nodoArticulo *head, int buscado);
struct articulo *crearArticulo(struct nodoArticulo *lista);
void agregarArticulo(struct congreso *congreso, struct nodoArticulo **lista);
int eliminarArticulo(struct nodoArticulo **lista, int seccionEliminada);
int modificarArticulo(struct nodoArticulo *articulos, int seccionModificada);
void funcionSwitch(char opcion, struct congreso *congreso, void (*submenu)(struct congreso *));
void menuProyectosLey(struct congreso *congreso);
void menuCongresistas(struct congreso *congreso);
void gestionarVotacionArticulo(struct congreso *congreso, struct nodoArticulo *articulos);
void mostrarVotacionArticulos(struct proyectoLey *ley);
void menuArticulos(struct congreso *congreso, struct proyectoLey *ley);
void menuComisiones(struct congreso *congreso);
void mostrarLeyesPorFase(struct nodoProyectoLey *leyes, int faseRequerida);
/*NOTA: LAS FUNCIONES DE VOTACIÓN SE ENCUENTRAN EN EL APARTADO DE LAS FUNCIONES DE PROYECTO DE LEY.*/


/*TODO: FUNCIONES AUXILIARES---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/
/*TODO---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/

void convertirMinusculas(char *cadena) {
    int i;
    for (i = 0; cadena[i]; i++) {
        if (cadena[i] >= 'A' && cadena[i] <= 'Z') {
            cadena[i] += 'a' - 'A';
        }
    }
}


int leerEnteroConLimite(char *mensaje, int min, int max) {
    int valor;
    char input[10];
    while (1) {
        printf("%s (%d-%d): ", mensaje, min, max);
        scanf("%9s", input);
        valor = atoi(input);
        if (valor >= min && valor <= max) {
            break;
        }
        printf("Error: Valor inválido. Debe estar entre %d y %d.\n", min, max);
    }
    return valor;
}

// Función auxiliar para buscar un congresista por RUT en el congreso
struct congresista *buscarCongresistaPorRUT(struct congreso *congreso, char *rut) {
    int i;

    // Buscar en el arreglo de senadores
    for (i = 0; congreso->senadores[i] != 0; i++) {
        if (strcmp(congreso->senadores[i]->rut, rut) == 0) {
            return congreso->senadores[i];
        }
    }

    // Buscar en el arreglo de diputados
    for (i = 0; congreso->diputados[i] != 0; i++) {
        if (strcmp(congreso->diputados[i]->rut, rut) == 0) {
            return congreso->diputados[i];
        }
    }

    // Si no se encuentra el congresista
    return 0;
}

void limpiarBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) {
        /* Mantener el bucle hasta encontrar un salto de línea o el fin de archivo */
    }
}

char leerOpcion() {
    char opcion;
    scanf("%c", &opcion);
    limpiarBuffer();  /* Llama a la función para limpiar el buffer */

    /* Convierte a mayúscula si es una letra minúscula */
    if (opcion >= 'a' && opcion <= 'z') {
        opcion -= 'a' - 'A';
    }

    return opcion;
}

//TODO: FUNCIÓN DE INICIALIZACIÓN DEL CONGRESO----------------------------------------------------------------------------------------------------------------------------------------------------//
//TODO: ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------//
/*Esta función almacena memoria para todos los datos que debe almacenar "congreso"*/
struct congreso *inicializarCongreso() {
    struct congreso *nuevoCongreso = malloc(sizeof(struct congreso));

    if (nuevoCongreso == NULL) {
        return NULL;
    }

    // Inicializa los arreglos para los senadores y diputados
    nuevoCongreso->senadores = calloc(MAX_SENADORES, sizeof(struct congresista *));
    nuevoCongreso->diputados = calloc(MAX_DIPUTADOS, sizeof(struct congresista *));
    nuevoCongreso->congresistasMixtos = NULL;
    nuevoCongreso->comisionesMixtas = NULL;

    if (nuevoCongreso->senadores == NULL || nuevoCongreso->diputados == NULL) {
        free(nuevoCongreso);
        return NULL;
    }

    // Inicializa los arreglos para las comisiones
    nuevoCongreso->comisiones = calloc(MAX_COMISIONES, sizeof(struct comision *));

    if (nuevoCongreso->comisiones == NULL) {
        free(nuevoCongreso->senadores);
        free(nuevoCongreso->diputados);
        free(nuevoCongreso);
        return NULL;
    }

    // Inicializa la raíz de proyectos de ley
    nuevoCongreso->raiz = NULL;
    return nuevoCongreso;
}

void liberarCongreso(struct congreso *congreso) {
    if (congreso == NULL) {
        return;
    }

    // Libera los arreglos de punteros si fueron asignados
    if (congreso->senadores != NULL) {
        free(congreso->senadores);
    }
    if (congreso->diputados != NULL) {
        free(congreso->diputados);
    }
    if (congreso->comisiones != NULL) {
        free(congreso->comisiones);
    }

    free(congreso);
}

//TODO: FUNCIÓNES DEL PROYECTO DE LEY----------------------------------------------------------------------------------------------------------------------------------------------------//
//TODO-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------//
struct nodoProyectoLey *crearNodoProyectoLey(struct proyectoLey *datos) {
    struct nodoProyectoLey *nodo = NULL;

    // Validar que los datos no sean NULL
    if (datos != NULL) {
        nodo = malloc(sizeof(struct nodoProyectoLey));

        if (nodo == NULL) {
            printf("Error: No se pudo asignar memoria para el nodo de proyecto de ley.\n");
            return NULL;
        }

        nodo->datos = datos; // Copiar los datos recibidos
        nodo->izq = NULL;
        nodo->der = NULL;
    }

    return nodo;
}

struct proyectoLey *crearProyectoLey(struct congreso *congreso) {
    // Asignar memoria para el proyecto de ley
    struct proyectoLey *nuevoProyecto = malloc(sizeof(struct proyectoLey));
    if (!nuevoProyecto) {
        printf("Error: No se pudo asignar memoria para el proyecto de ley.\n");
        return NULL;
    }

    char nombre[100], tipo[50];
    int idProyecto, urgencia, fase;

    // Capturar datos de texto con espacios
    printf("Ingrese el nombre del proyecto de ley: ");
    if (fgets(nombre, sizeof(nombre), stdin) == NULL) {
        printf("Error al leer el nombre del proyecto de ley.\n");
        free(nuevoProyecto);
        return NULL;
    }
    if (nombre[strlen(nombre) - 1] != '\n') {
        limpiarBuffer();
    } else {
        nombre[strcspn(nombre, "\n")] = '\0';
    }

    printf("Ingrese el tipo del proyecto de ley: ");
    if (fgets(tipo, sizeof(tipo), stdin) == NULL) {
        printf("Error al leer el tipo del proyecto de ley.\n");
        free(nuevoProyecto);
        return NULL;
    }
    if (tipo[strlen(tipo) - 1] != '\n') {
        limpiarBuffer();
    } else {
        tipo[strcspn(tipo, "\n")] = '\0';
    }

    // Validación y captura del ID con verificación de duplicados
    do {
        idProyecto = leerEnteroConLimite("Ingrese el ID del proyecto", 10000000, 99999999);
        if (buscarProyectoLeyPorID(congreso->raiz, idProyecto)) {
            printf("Error: El ID ya está en uso. Ingrese otro ID.\n");
        }
    } while (buscarProyectoLeyPorID(congreso->raiz, idProyecto));

    urgencia = leerEnteroConLimite("Ingrese la urgencia del proyecto", 1, 5);
    fase = leerEnteroConLimite("Ingrese la fase del proyecto", 1, 8);

    // Asignar memoria para campos de texto y verificar
    nuevoProyecto->nombre = malloc(strlen(nombre) + 1);
    nuevoProyecto->tipo = malloc(strlen(tipo) + 1);
    if (!nuevoProyecto->nombre || !nuevoProyecto->tipo) {
        printf("Error: No se pudo asignar memoria para los datos del proyecto de ley.\n");
        free(nuevoProyecto->nombre);
        free(nuevoProyecto->tipo);
        free(nuevoProyecto);
        return NULL;
    }

    // Copiar datos y completar estructura
    strcpy(nuevoProyecto->nombre, nombre);
    strcpy(nuevoProyecto->tipo, tipo);
    nuevoProyecto->idProyecto = idProyecto;
    nuevoProyecto->urgencia = urgencia;
    nuevoProyecto->fase = fase;

    // Inicializar punteros a NULL
    nuevoProyecto->articulo = NULL;
    nuevoProyecto->votacion = NULL;
    nuevoProyecto->comision = NULL;

    return nuevoProyecto;
}


// Función para añadir un nodo al árbol binario de búsqueda
void agregarNodoProyectoLey(struct congreso *congreso) {
    struct proyectoLey *datos = crearProyectoLey(congreso); // Crear proyecto de ley
    if (datos == NULL) {
        printf("Error: No se pudo crear el proyecto de ley.\n");
        return;
    }

    struct nodoProyectoLey *nuevoNodo = crearNodoProyectoLey(datos); // Crear nodo para el proyecto de ley
    if (nuevoNodo == NULL) {
        printf("Error: No se pudo crear el nodo del proyecto de ley.\n");
        free(datos); // Liberar datos si no se pudo crear el nodo
        return;
    }

    if (congreso->raiz == NULL) {
        // El árbol binario de búsqueda no existe, entonces el nuevo nodo será la raíz
        congreso->raiz = nuevoNodo;
    } else {
        struct nodoProyectoLey *actual = congreso->raiz;
        struct nodoProyectoLey *padre = NULL;

        // Procedimiento estándar para añadir nodos a un árbol binario de búsqueda
        while (actual != NULL) {
            padre = actual;
            if (datos->idProyecto < actual->datos->idProyecto) {
                actual = actual->izq;
            } else {
                actual = actual->der;
            }
        }

        // Añadir el nuevo nodo como hijo del nodo padre adecuado
        if (datos->idProyecto < padre->datos->idProyecto) {
            padre->izq = nuevoNodo;
        } else {
            padre->der = nuevoNodo;
        }
    }
}

// Función para buscar un proyecto de ley por ID
struct proyectoLey *buscarProyectoLeyPorID(struct nodoProyectoLey *raiz, int id) {
    if (raiz == NULL) {
        return NULL;
    }

    if (raiz->datos->idProyecto == id) {
        return raiz->datos;
    } else if (id < raiz->datos->idProyecto) {
        // Buscamos en el subárbol izquierdo si el ID es menor
        return buscarProyectoLeyPorID(raiz->izq, id);
    } else {
        // Buscamos en el subárbol derecho si el ID es mayor
        return buscarProyectoLeyPorID(raiz->der, id);
    }
}

// Encuentra el nodo mínimo en el árbol (usado para encontrar el sucesor en caso de eliminación)
struct nodoProyectoLey *minValorNodo(struct nodoProyectoLey *nodo) {
    struct nodoProyectoLey *actual = nodo;

    /* Recorre el subárbol hacia la izquierda para encontrar el nodo mínimo */
    while (actual != NULL && actual->izq != NULL) {
        actual = actual->izq;
    }
    return actual;
}

// Función para borrar un nodo en el árbol binario de búsqueda
struct nodoProyectoLey *borrarNodoProyectoLey(struct congreso *congreso, struct nodoProyectoLey *raiz, int id) {
    struct nodoProyectoLey *temp;

    /* Caso base */
    if (raiz == NULL) return raiz;

    /* Si el id a eliminar es más pequeño que el id de la raíz, está en el subárbol izquierdo */
    if (id < raiz->datos->idProyecto) {
        raiz->izq = borrarNodoProyectoLey(congreso, raiz->izq, id);
    }
    /* Si el id a eliminar es mayor que el id de la raíz, está en el subárbol derecho */
    else if (id > raiz->datos->idProyecto) {
        raiz->der = borrarNodoProyectoLey(congreso, raiz->der, id);
    }
    /* Si el id es el mismo que el id de la raíz, este es el nodo a eliminar */
    else {
        /* Nodo con solo un hijo o sin hijos */
        if (raiz->izq == NULL) {
            temp = raiz->der;
            free(raiz->datos->nombre);
            free(raiz->datos->tipo);
            free(raiz->datos);
            free(raiz);
            return temp;
        } else if (raiz->der == NULL) {
            temp = raiz->izq;
            free(raiz->datos->nombre);
            free(raiz->datos->tipo);
            free(raiz->datos);
            free(raiz);
            return temp;
        }

        /* Nodo con dos hijos: obtener el sucesor en el orden (el más pequeño en el subárbol derecho) */
        temp = minValorNodo(raiz->der);

        /* Copiar el contenido del sucesor al nodo actual */
        raiz->datos->idProyecto = temp->datos->idProyecto;

        /* Asegurarse de que haya suficiente memoria para copiar los campos */
        raiz->datos->nombre = realloc(raiz->datos->nombre, strlen(temp->datos->nombre) + 1);
        raiz->datos->tipo = realloc(raiz->datos->tipo, strlen(temp->datos->tipo) + 1);
        if (raiz->datos->nombre != NULL && raiz->datos->tipo != NULL) {
            strcpy(raiz->datos->nombre, temp->datos->nombre);
            strcpy(raiz->datos->tipo, temp->datos->tipo);
        }
        raiz->datos->urgencia = temp->datos->urgencia;
        raiz->datos->fase = temp->datos->fase;

        /* Borrar el sucesor */
        raiz->der = borrarNodoProyectoLey(congreso, raiz->der, temp->datos->idProyecto);
    }

    return raiz;
}

// Función para borrar un proyecto de ley
void borrarProyectoLey(struct congreso *congreso, int id) {
    congreso->raiz = borrarNodoProyectoLey(congreso, congreso->raiz, id);
}

// Función auxiliar para contar y mostrar los congresistas en una lista de votación
void mostrarCongresistasVotacion(struct nodoCongresista *lista, const char *categoria) {
    int contador = 0;
    struct nodoCongresista *actual = lista;

    printf("Votos %s:\n", categoria);
    while (actual != NULL) {
        printf("- %s (RUT: %s)\n", actual->datos->nombre, actual->datos->rut);
        contador++;
        actual = actual->sig;
    }
    printf("Total de votos %s: %d\n", categoria, contador);
}

// Función para buscar y mostrar un proyecto de ley por ID en el árbol binario de búsqueda
void buscarYMostrarProyectoLey(struct congreso *congreso, int id) {
    struct proyectoLey *proyecto = buscarProyectoLeyPorID(congreso->raiz, id);
    struct nodoArticulo *rec;

    if (proyecto != NULL) {
        printf("Nombre: %s\n", proyecto->nombre);
        printf("Tipo: %s\n", proyecto->tipo);
        printf("ID Proyecto: %d\n", proyecto->idProyecto);
        printf("Urgencia: %d\n", proyecto->urgencia);
        printf("Fase: %d\n", proyecto->fase);

        // Descripción de la fase dependiendo del caso
        switch (proyecto->fase) {
            case 1:
                printf("Descripción de la fase: Iniciativa Legislativa\n");
                break;
            case 2:
                printf("Descripción de la fase: Cámara de Origen\n");
                break;
            case 3:
                printf("Descripción de la fase: Cámara Revisora\n");
                break;
            case 4:
                printf("Descripción de la fase: Comisión Mixta\n");
                break;
            case 5:
                printf("Descripción de la fase: Promulgación\n");
                break;
            case 6:
                printf("Descripción de la fase: Veto presidencial\n");
                break;
            case 7:
                printf("Descripción de la fase: Publicación y vigencia\n");
                break;
            case 8:
                printf("Descripción de la fase: Control constitucional\n");
                break;
            default:
                printf("Fase desconocida.\n");
                break;
        }

        rec = proyecto->articulo;
        while (rec != NULL) {
            // Verificar que los datos del artículo no sean NULL antes de acceder
            if (rec->datos != NULL) {
                printf("Sección de artículo: %d\n", rec->datos->seccion);

                // Mostrar el texto del artículo si no es NULL
                if (rec->datos->texto != NULL) {
                    printf("Texto del artículo: %s\n", rec->datos->texto);
                } else {
                    printf("Texto del artículo: (sin texto disponible)\n");
                }
            } else {
                printf("Artículo no disponible.\n");
            }
            rec = rec->sig; // Avanzar al siguiente nodo
        }

        // Verificar si hay votaciones y mostrarlas
        if (proyecto->votacion != NULL) {
            struct nodoVotacion *nodoVot = proyecto->votacion;

            // Recorre la lista de votaciones
            while (nodoVot != NULL) {
                printf("\nFase de votación: %d\n", nodoVot->datos->fase);

                // Mostrar congresistas a favor y contar votos
                mostrarCongresistasVotacion(nodoVot->datos->favor, "a favor");

                // Mostrar congresistas en contra y contar votos
                mostrarCongresistasVotacion(nodoVot->datos->contra, "en contra");

                nodoVot = nodoVot->sig; // Avanzar a la siguiente votación
            }
        } else {
            printf("No hay votaciones registradas para este proyecto de ley.\n");
        }
    } else {
        printf("Proyecto de ley con ID %d no encontrado.\n", id);
    }
}

// Función para limpiar el buffer de entrada en caso de entrada inválida
void limpiarBufferEntrada() {
    int c; // Variable para almacenar el carácter leído
    // Leer y descartar caracteres hasta encontrar un salto de línea o EOF
    while ((c = getchar()) != '\n' && c != EOF);
}

// Función para añadir congresistas a las listas de votos (a favor o en contra) en un nodo de votación
void agregarCongresistaAVotacion(struct votacion *votacion, struct congreso *congreso) {
    int opcionLista, opcionAgregar;
    char rut[20];
    struct congresista *congresista;
    struct nodoCongresista *nuevoNodo;
    struct nodoCongresista *actual;

    // Elegir la lista de votación a la que se desea agregar congresistas
    printf("Seleccione la lista de votación:\n");
    printf("1. A favor\n");
    printf("2. En contra\n");
    printf("Ingrese su opción: ");

    while (scanf("%d", &opcionLista) != 1 || (opcionLista != 1 && opcionLista != 2)) {
        printf("Opción inválida. Seleccione 1 (A favor) o 2 (En contra): ");
        limpiarBufferEntrada(); // Limpia el buffer si el input no es válido
    }
    limpiarBufferEntrada(); // Limpia el buffer tras una entrada válida

    while (1) {
        printf("\nSeleccione una opción:\n");
        printf("1. Agregar congresista\n");
        printf("2. Terminar\n");
        printf("Ingrese su opción: ");

        while (scanf("%d", &opcionAgregar) != 1 || (opcionAgregar != 1 && opcionAgregar != 2)) {
            printf("Opción inválida. Intente de nuevo.\n");
            limpiarBufferEntrada(); // Limpia el buffer si el input no es válido
        }
        limpiarBufferEntrada(); // Limpia el buffer tras una entrada válida

        switch (opcionAgregar) {
            case 1: // Agregar congresista
                printf("¿Qué congresista añadiremos a la lista? Ingrese el RUT: ");
                fgets(rut, sizeof(rut), stdin);
                size_t len = strlen(rut);
                if (len > 0 && rut[len - 1] == '\n') {
                    rut[len - 1] = '\0'; // Elimina el salto de línea al final del RUT
                }

                // Buscar el congresista en el congreso
                congresista = comprobarCongresistaEnCongreso(congreso, rut);
                if (congresista == NULL) {
                    printf("Error: Congresista con RUT %s no encontrado.\n", rut);
                    continue;
                }

                // Crear un nuevo nodoCongresista
                nuevoNodo = (struct nodoCongresista *)malloc(sizeof(struct nodoCongresista));
                if (nuevoNodo == NULL) {
                    printf("Error: No se pudo asignar memoria para el nuevo nodo de congresista.\n");
                    return;
                }
                nuevoNodo->datos = congresista;
                nuevoNodo->sig = NULL;

                // Añadir el nuevo nodo a la lista seleccionada
                if (opcionLista == 1) { // Lista de votos a favor
                    if (votacion->favor == NULL) {
                        votacion->favor = nuevoNodo;
                    } else {
                        actual = votacion->favor;
                        while (actual->sig != NULL) {
                            actual = actual->sig;
                        }
                        actual->sig = nuevoNodo;
                    }
                    printf("Congresista con RUT %s añadido correctamente a la lista de votos a favor.\n", rut);
                } else if (opcionLista == 2) { // Lista de votos en contra
                    if (votacion->contra == NULL) {
                        votacion->contra = nuevoNodo;
                    } else {
                        actual = votacion->contra;
                        while (actual->sig != NULL) {
                            actual = actual->sig;
                        }
                        actual->sig = nuevoNodo;
                    }
                    printf("Congresista con RUT %s añadido correctamente a la lista de votos en contra.\n", rut);
                }
                break;

            case 2: // Terminar
                printf("Finalizando el agregado de congresistas.\n");
                return;

            default:
                printf("Opción inválida. Intente de nuevo.\n");
                break;
        }
    }
}

void agregarVotacion(struct congreso *congreso, int idProyecto) {
    struct proyectoLey *proyecto;
    int fase;
    struct nodoVotacion *nuevoNodoVotacion, *actual;

    // Buscar el proyecto de ley por ID
    proyecto = buscarProyectoLeyPorID(congreso->raiz, idProyecto);
    if (proyecto == NULL) {
        printf("Error: Proyecto con ID %d no encontrado.\n", idProyecto);
        return;
    }

    // Solicitar y verificar la fase de la votación
    fase = leerEnteroConLimite("Ingrese la fase de la votación", 1, 4);

    // Crear nuevo nodoVotacion
    nuevoNodoVotacion = (struct nodoVotacion *)malloc(sizeof(struct nodoVotacion));
    if (nuevoNodoVotacion == NULL) {
        printf("Error: No se pudo asignar memoria para la nueva votación.\n");
        return;
    }

    // Crear el struct votacion y asignar datos
    nuevoNodoVotacion->datos = (struct votacion *)malloc(sizeof(struct votacion));
    if (nuevoNodoVotacion->datos == NULL) {
        printf("Error: No se pudo asignar memoria para los datos de votación.\n");
        free(nuevoNodoVotacion);
        return;
    }
    nuevoNodoVotacion->datos->fase = fase;
    nuevoNodoVotacion->datos->favor = NULL;   // Inicializar lista de votantes a favor
    nuevoNodoVotacion->datos->contra = NULL;  // Inicializar lista de votantes en contra
    nuevoNodoVotacion->sig = NULL;

    // Llamar a agregarCongresistaAVotacion para rellenar las listas de favor y en contra
    agregarCongresistaAVotacion(nuevoNodoVotacion->datos, congreso);

    // Agregar el nuevo nodo al final de la lista de votaciones del proyecto
    if (proyecto->votacion == NULL) {
        // Caso 1: Primera votación para este proyecto
        proyecto->votacion = nuevoNodoVotacion;
    } else {
        // Caso 2: Ya existen votaciones, recorrer hasta el final de la lista
        actual = proyecto->votacion;
        while (actual->sig != NULL) {
            actual = actual->sig;
        }
        actual->sig = nuevoNodoVotacion;
    }

    printf("Votación añadida correctamente al proyecto con ID %d.\n", idProyecto);
}

// Función auxiliar para imprimir un proyecto de ley
void imprimirProyectoLey(struct proyectoLey *proyecto) {
    if (proyecto == NULL) {
        return;
    }

    printf("Nombre: %s\n", proyecto->nombre ? proyecto->nombre : "N/A");
    printf("Tipo: %s\n", proyecto->tipo ? proyecto->tipo : "N/A");
    printf("ID Proyecto: %d\n", proyecto->idProyecto);
    printf("Urgencia: %d\n", proyecto->urgencia);
    printf("Fase: %d\n", proyecto->fase);

    // Verificar e imprimir campos opcionales
    if (proyecto->articulo != NULL) {
        printf("Artículo: (existe artículo)\n"); // Puedes detallar más si tienes la estructura
        // Aquí podrías imprimir detalles específicos del artículo
        // printf("Sección: %d\n", proyecto->articulo->seccion);
        // printf("Texto: %s\n", proyecto->articulo->texto ? proyecto->articulo->texto : "N/A");
    } else {
        printf("Artículo: N/A\n");
    }

    if (proyecto->votacion != NULL) {
        printf("Votación: (existe votación)\n"); // Detallar según la estructura
    } else {
        printf("Votación: N/A\n");
    }

    if (proyecto->comision != NULL) {
        printf("Comisión: (existe comisión)\n"); // Detallar según la estructura
    } else {
        printf("Comisión: N/A\n");
    }

    printf("----\n");
}

// Función recursiva para recorrer e imprimir los proyectos de ley en el árbol
void recorrerYImprimirProyectos(struct nodoProyectoLey *nodo) {
    if (nodo == NULL) {
        return;
    }

    // Recorrer el subárbol izquierdo
    recorrerYImprimirProyectos(nodo->izq);

    // Imprimir el proyecto de ley en el nodo actual
    imprimirProyectoLey(nodo->datos);

    // Recorrer el subárbol derecho
    recorrerYImprimirProyectos(nodo->der);
}

// Función principal para imprimir todos los proyectos de ley del árbol
void imprimirProyectosLey(struct congreso *congreso) {
    if (congreso == NULL || congreso->raiz == NULL) {
        printf("No hay proyectos de ley para mostrar.\n");
        return;
    }

    // Llamar a la función que recorre e imprime los proyectos de ley
    recorrerYImprimirProyectos(congreso->raiz);
}

void agregarProyecto(struct proyectoLey *proyecto, struct proyectoLey ***proyectosArray, int *proyectosCount, int *proyectosCapacidad) {
    struct proyectoLey **tempArray;

    // Verificar si se necesita redimensionar el array
    if (*proyectosCount >= *proyectosCapacidad) {
        *proyectosCapacidad *= 2; // Duplicar la capacidad
        tempArray = (struct proyectoLey **) realloc(*proyectosArray, (*proyectosCapacidad) * sizeof(struct proyectoLey *));
        if (tempArray == NULL) {
            printf("Error al redimensionar la memoria.\n");
            exit(1); // Terminar el programa en caso de fallo en realloc
        }
        *proyectosArray = tempArray; // Actualizar el puntero al nuevo array
    }

    // Agregar el nuevo proyecto al array y aumentar el contador
    (*proyectosArray)[(*proyectosCount)++] = proyecto;
}

// Recorrido en orden del árbol binario para agregar los proyectos a la lista
void recorrerArbolEnOrden(struct nodoProyectoLey *nodo, struct proyectoLey ***proyectosArray, int *proyectosCount, int *proyectosCapacidad) {
    if (nodo == NULL) {
        return; // Caso base: si el nodo es NULL, salir de la función
    }

    // Recorrer el subárbol izquierdo
    recorrerArbolEnOrden(nodo->izq, proyectosArray, proyectosCount, proyectosCapacidad);

    // Agregar el proyecto del nodo actual al array
    agregarProyecto(nodo->datos, proyectosArray, proyectosCount, proyectosCapacidad);

    // Recorrer el subárbol derecho
    recorrerArbolEnOrden(nodo->der, proyectosArray, proyectosCount, proyectosCapacidad);
}

// Función de comparación para ordenar los proyectos por urgencia (mayor a menor)
int compararPorUrgencia(const void *a, const void *b) {
    struct proyectoLey *proyectoA = *(struct proyectoLey **)a; // Convertir el primer argumento
    struct proyectoLey *proyectoB = *(struct proyectoLey **)b; // Convertir el segundo argumento
    return proyectoB->urgencia - proyectoA->urgencia; // Comparar por urgencia
}

// Función principal para mostrar los proyectos en orden de urgencia
void mostrarProyectosOrdenDeUrgencia(struct congreso *congreso) {
    struct proyectoLey **proyectosArray;
    int proyectosCount = 0;
    int proyectosCapacidad = 10;
    int i;
    struct proyectoLey *proyecto;

    // Inicializar el array dinámico de proyectos
    proyectosArray = (struct proyectoLey **) malloc(proyectosCapacidad * sizeof(struct proyectoLey *));
    if (proyectosArray == NULL) {
        printf("Error al asignar memoria.\n");
        exit(1); // Terminar el programa en caso de fallo en malloc
    }

    // Recorrer el árbol y recolectar los proyectos
    recorrerArbolEnOrden(congreso->raiz, &proyectosArray, &proyectosCount, &proyectosCapacidad);

    // Ordenar los proyectos por urgencia de mayor a menor
    qsort(proyectosArray, proyectosCount, sizeof(struct proyectoLey *), compararPorUrgencia);

    // Imprimir los proyectos en el orden de urgencia
    printf("Proyectos de ley en orden de urgencia (de mayor a menor):\n");
    for (i = 0; i < proyectosCount; i++) {
        proyecto = proyectosArray[i];
        printf("Nombre: %s\n", proyecto->nombre ? proyecto->nombre : "N/A");
        printf("Tipo: %s\n", proyecto->tipo ? proyecto->tipo : "N/A");
        printf("ID Proyecto: %d\n", proyecto->idProyecto);
        printf("Urgencia: %d\n", proyecto->urgencia);
        printf("\n");
    }

    // Liberar la memoria del array dinámico
    free(proyectosArray);
}

void modificarProyectoLey(struct congreso *congreso, int idProyecto) {
    struct proyectoLey *proyecto;
    char nombreComision[100]; // Para el nombre de la comisión
    struct comision *comisionEncontrada = NULL; // Para el puntero de comisión
    char opcion;

    // Buscar el proyecto de ley por ID
    proyecto = buscarProyectoLeyPorID(congreso->raiz, idProyecto);
    if (!proyecto) {
        printf("Error: Proyecto de ley no encontrado.\n");
        return;
    }

    do {
        printf("Seleccione el campo a modificar:\n");
        printf("a. Nombre\n");
        printf("b. Tipo\n");
        printf("c. ID Proyecto\n");
        printf("d. Urgencia\n");
        printf("e. Fase\n");
        printf("f. Agregar votación\n");
        printf("g. Gestionar Artículos\n");
        printf("h. Asignar Comisión\n");
        printf("i. Salir\n");
        printf("Opción: ");

        // Leer opción y limpiar buffer de entrada
        scanf(" %c", &opcion);
        limpiarBufferEntrada(); // Función para limpiar el buffer

        // Convertir a minúscula si es una letra mayúscula
        if (opcion >= 'A' && opcion <= 'Z') {
            opcion += 'a' - 'A';
        }

        switch (opcion) {
            case 'a': {
                char nuevoNombre[100];
                printf("Ingrese nuevo nombre: ");
                if (fgets(nuevoNombre, sizeof(nuevoNombre), stdin) == NULL) {
                    printf("Error al leer el nombre.\n");
                    break;
                }
                nuevoNombre[strcspn(nuevoNombre, "\n")] = '\0'; // Remover salto de línea
                free(proyecto->nombre);
                proyecto->nombre = (char *)malloc(strlen(nuevoNombre) + 1);
                if (proyecto->nombre) {
                    strcpy(proyecto->nombre, nuevoNombre);
                } else {
                    printf("Error: No se pudo asignar memoria para el nuevo nombre.\n");
                }
                break;
            }
            case 'b': {
                char nuevoTipo[50];
                printf("Ingrese nuevo tipo: ");
                if (fgets(nuevoTipo, sizeof(nuevoTipo), stdin) == NULL) {
                    printf("Error al leer el tipo.\n");
                    break;
                }
                nuevoTipo[strcspn(nuevoTipo, "\n")] = '\0'; // Remover salto de línea
                free(proyecto->tipo);
                proyecto->tipo = (char *)malloc(strlen(nuevoTipo) + 1);
                if (proyecto->tipo) {
                    strcpy(proyecto->tipo, nuevoTipo);
                } else {
                    printf("Error: No se pudo asignar memoria para el nuevo tipo.\n");
                }
                break;
            }
            case 'c': {
                int nuevoID = leerEnteroConLimite("Ingrese nuevo ID de proyecto", 10000000, 99999999);
                proyecto->idProyecto = nuevoID;
                break;
            }
            case 'd': {
                int nuevaUrgencia = leerEnteroConLimite("Ingrese nueva urgencia", 1, 5);
                proyecto->urgencia = nuevaUrgencia;
                break;
            }
            case 'e': {
                int nuevaFase = leerEnteroConLimite("Ingrese nueva fase", 1, 8);
                proyecto->fase = nuevaFase;
                break;
            }
            case 'f':
                // Llamar a la función para agregar votación al proyecto de ley
                agregarVotacion(congreso, idProyecto);
                break;
            case 'g':
                // Llamar al menú de artículos
                menuArticulos(congreso, proyecto);
                break;
            case 'h':
                printf("Ingrese el nombre de la comisión a asignar: ");
                if (fgets(nombreComision, sizeof(nombreComision), stdin) == NULL) {
                    printf("Error al leer el nombre de la comisión.\n");
                    break;
                }
                nombreComision[strcspn(nombreComision, "\n")] = '\0'; // Remover salto de línea
                comisionEncontrada = buscarComision(congreso, nombreComision); // IMPORTANTE INICIALIZAR FUNCION BUSCARCOMISION
                if (comisionEncontrada) {
                    proyecto->comision = comisionEncontrada;
                    printf("Comisión asignada correctamente\n");
                } else {
                    printf("Error: Comisión no asignada\n");
                }
                break;
            case 'i':
                printf("Saliendo de la modificación del proyecto de ley.\n");
                break;
            default:
                printf("Opción inválida. Intente nuevamente.\n");
        }
    } while (opcion != 'i');
}

/*TODO: FUNCIONES DE CONGRESISTAS------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/
/*TODO------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/
struct nodoCongresista *crearNodoCongresista(struct nodoCongresista *head, struct congresista *datos) {
    struct nodoCongresista *nodo = NULL;

    // Pregunto primero que los datos recibidos no sean NULL
    if (datos != NULL) {
        nodo = (struct nodoCongresista *)malloc(sizeof(struct nodoCongresista));

        if (nodo == NULL) {
            // Si esto ocurre, hay un error al asignar la memoria
            return NULL;
        }

        nodo->datos = datos; // Copio los datos que recibí
        nodo->sig = NULL; // Asigno NULL al siguiente nodo
    }

    return nodo; // Retorno el nuevo nodo (o NULL si no se creó)
}

/*

comprobar que exista congresista, lo haré de manera que retorne 0 si NO existe el RUT, o que retorne 1 si existe
la idea es que el RUT sea el buscado, por lo tanto las otras funciones que la llamen deben ingresar el RUT
aunque esto puede estar sujeto a cambios si se desea, quizas recibir el nodo entero para comodidad
recordar que es circular con fantasma, por lo tanto tengo que iniciar el head->sig para el rec y usar do while

*/

int comprobarCongresistaEnComision(struct nodoCongresista *head, char *rutBuscado) {
    struct nodoCongresista *rec = head; // Iniciamos en el nodo fantasma

    if (head == NULL || head->sig == NULL) {
        return 0; // Si la lista está vacía, retorna 0
    }

    rec = head->sig; // Saltar el nodo fantasma
    do {
        // Verificamos que rec->datos no sea NULL y comparamos el RUT
        if (rec->datos != NULL && strcmp(rec->datos->rut, rutBuscado) == 0) {
            return 1; // Se encontró el congresista en la lista
        }
        rec = rec->sig; // Avanzar al siguiente nodo
    } while (rec != head); // Volver al nodo fantasma

    return 0; // No se encontró el congresista en la lista
}

//funcion para recorrer los arreglos, el de diputados o el de senadores correspondientemente
struct congresista* comprobarCongresistaEnCongreso(struct congreso *congreso, char *rutBuscado) {
    int i;
    struct nodoCongresista *head = congreso->congresistasMixtos;
    struct nodoCongresista *rec = NULL;

    if (rutBuscado != NULL) {
        // Buscar en diputados
        for (i = 0; i < MAX_DIPUTADOS; i++) {
            if (congreso->diputados[i] != NULL) {
                if (strcmp(congreso->diputados[i]->rut, rutBuscado) == 0) {
                    return congreso->diputados[i]; // RUT encontrado
                }
            }
        }

        // Buscar en senadores
        for (i = 0; i < MAX_SENADORES; i++) {
            if (congreso->senadores[i] != NULL) {
                if (strcmp(congreso->senadores[i]->rut, rutBuscado) == 0) {
                    return congreso->senadores[i]; // RUT encontrado
                }
            }
        }

        // Buscar en la lista de congresistas mixtos
        if (head != NULL) {
            rec = head; // Iniciar en la cabeza de la lista
            while (rec != NULL) {
                if (rec->datos != NULL && strcmp(rec->datos->rut, rutBuscado) == 0) {
                    return rec->datos; // RUT encontrado
                }
                rec = rec->sig; // Avanzar al siguiente nodo
            }
        }
    }
    return NULL; // RUT no encontrado
}
/*
El crearCongresista va a ser un poco distinto, esta funcion tiene que recibir(scanf) el rut y la ocupacion
al saber la ocupacion eligirá que arreglo recorrer si el de senadores o el de diputados
ahí en ese momento buscará comparando el rut en cada pos del arreglo, si lo encuentra retornará 1
si no es así retornará 0 y dará paso a la copia de datos para ingresarlos en el arreglo

aunque no estoy seguro si la ocupacion fuese el diputado/senador o si es la especializacion

*/

struct congresista *crearCongresista(struct congreso *congreso) {
    struct congresista *nuevoCongresista; // Para guardar los datos del congresista
    char nombre[100], rut[20], ocupacion[20], especializacion[100]; // Datos que se reciben

    nuevoCongresista = (struct congresista *) malloc(sizeof(struct congresista));
    if (nuevoCongresista == NULL) {
        return NULL; // Error al asignar memoria
    }

    // Tomar el RUT del congresista
    printf("Ingresa el RUT del congresista (12.345.678-9): ");
    scanf("%19[^\n]", rut);
    limpiarBuffer();

    if (strlen(rut) > 20) {
        printf("El RUT ingresado es muy largo, inténtelo de nuevo\n");
        limpiarBuffer();
        free(nuevoCongresista); // Liberar memoria asignada
        return NULL;
    }

    // Ingresar la ocupación
    printf("Ingrese la ocupación (senador/diputado): ");
    scanf("%19s", ocupacion);
    limpiarBuffer();

    if (comprobarCongresistaEnCongreso(congreso, rut) != NULL) {
        printf("Este congresista ya se encuentra en el sistema\n");
        free(nuevoCongresista); // Liberar memoria asignada
        return NULL; // El congresista ya existe
    }

    // Escanear los últimos datos
    printf("Ingrese su especialización: ");
    scanf("%[^\n]", especializacion);
    limpiarBuffer();

    printf("Ingrese su Nombre: ");
    scanf("%[^\n]", nombre);
    limpiarBuffer();

    if (strlen(rut) > 20 || strlen(especializacion) > 100 || strlen(nombre) > 100) {
        printf("Uno de los valores ingresados es muy largo, inténtelo otra vez\n");
        free(nuevoCongresista); // Liberar memoria asignada
        return NULL;
    }

    // Asignación de memoria para los campos de texto
    nuevoCongresista->nombre = (char *)malloc(sizeof(char) * (strlen(nombre) + 1));
    nuevoCongresista->rut = (char *)malloc(sizeof(char) * (strlen(rut) + 1));
    nuevoCongresista->ocupacion = (char *)malloc(sizeof(char) * (strlen(ocupacion) + 1));
    nuevoCongresista->especializacion = (char *)malloc(sizeof(char) * (strlen(especializacion) + 1));

    // Verificación de asignación de memoria
    if (nuevoCongresista->nombre == NULL || nuevoCongresista->rut == NULL ||
        nuevoCongresista->ocupacion == NULL || nuevoCongresista->especializacion == NULL) {
        free(nuevoCongresista->nombre);
        free(nuevoCongresista->rut);
        free(nuevoCongresista->ocupacion);
        free(nuevoCongresista->especializacion);
        free(nuevoCongresista); // Liberar memoria asignada
        return NULL; // Error de asignación de memoria
    }

    // Copiar los datos a los campos
    strcpy(nuevoCongresista->nombre, nombre);
    strcpy(nuevoCongresista->rut, rut);
    strcpy(nuevoCongresista->ocupacion, ocupacion);
    strcpy(nuevoCongresista->especializacion, especializacion);

    printf("Se ha agregado con éxito\n");
    return nuevoCongresista;
}

/*
primero hago la funcion para insertar el nuevo congresista en el arreglo correspondiente

agregue un MAX_CONGRESISTAS = 200 por que son arreglos separados
al ver la historia de chile te das cuenta que nunca han habido mas de 150 diputados
y tampoco han habido mas de 50 senadores en periodos de mas de 200 años
por lo tanto dudo que en otros 100 años se superen los 200
era esta solución o configurar el struct congreso para agregar un plibre de cada arreglo

*/

//este agrega un NUEVO congresista al arreglo correspondiente
void agregarCongresistaEnCongreso(struct congreso *congreso) {
    int i = 0;
    struct congresista *nuevoCongresista = crearCongresista(congreso); // Se crea el congresista para insertarlo
    struct nodoCongresista *nuevoNodoCongresista = NULL; // Nodo para congresistas mixtos
    struct congresista **arreglo = NULL; // Decidirá a qué arreglo pertenece

    // Preguntar si el congresista es válido
    if (nuevoCongresista != NULL) {
        if (strcmp(nuevoCongresista->ocupacion, "senador") == 0) {
            arreglo = congreso->senadores; // El congresista es un senador
        } else if (strcmp(nuevoCongresista->ocupacion, "diputado") == 0) {
            arreglo = congreso->diputados; // El congresista es un diputado
        } else {
            nuevoNodoCongresista = crearNodoCongresista(congreso->congresistasMixtos, nuevoCongresista);
            if (nuevoNodoCongresista == NULL) {
                printf("Error al asignar memoria\n");
                free(nuevoCongresista); // Liberar memoria si hay un error
                return; // Error al asignar memoria
            }
            nuevoNodoCongresista->datos = nuevoCongresista;
            nuevoNodoCongresista->sig = congreso->congresistasMixtos;
            congreso->congresistasMixtos = nuevoNodoCongresista;

            return; // Se agrega correctamente
        }

        // Se recorre el arreglo elegido validando que se haya elegido uno
        if (arreglo != NULL) {
            if (arreglo == congreso->senadores) {
                while (i < MAX_SENADORES && arreglo[i] != NULL) {
                    i++; // Se busca la primera posición vacía
                }
                if (i < MAX_SENADORES) {
                    arreglo[i] = nuevoCongresista; // Añade el nuevo congresista
                    return; // Se logra asignar sin problema
                } else {
                    printf("El arreglo está lleno, no se ha agregado\n");
                    free(nuevoCongresista); // Liberar memoria si no se agrega
                    return;
                }
            }

            if (arreglo == congreso->diputados) {
                while (i < MAX_DIPUTADOS && arreglo[i] != NULL) {
                    i++; // Se busca la primera posición vacía
                }
                if (i < MAX_DIPUTADOS) {
                    arreglo[i] = nuevoCongresista; // Añade el nuevo congresista
                    return; // Se logra asignar sin problema
                } else {
                    printf("El arreglo está lleno, no se ha agregado\n");
                    free(nuevoCongresista); // Liberar memoria si no se agrega
                    return;
                }
            }
        }
    }
    // El nuevo congresista era NULL
    printf("Error: No se pudo crear el nuevo congresista.\n");
}

/*
esta funcion agrega un ya EXISTENTE congresista a la lista de comisiones
voy a darle la comision, pues para acceder a la lista simplemente hago comision->headintegrantes
esto puede cambiar por int para los print
*/

void agregarCongresistaEnComision(struct comision *comision, char *rut, struct congresista *congresista) {
    struct nodoCongresista *nuevoNodo = NULL;
    struct nodoCongresista *rec = NULL;

    // Preguntar que ni uno de los dos sea NULL
    if (comision != NULL && congresista != NULL) {
        // Verifico que el congresista no esté en la comisión
        if (comprobarCongresistaEnComision(comision->headIntegrantes, rut) == 0) {
            // Creo y enlazo el nuevo nodo
            nuevoNodo = crearNodoCongresista(comision->headIntegrantes, congresista);
            if (nuevoNodo == NULL) {
                return; // Error al crear el nuevo nodo
            }

            rec = comision->headIntegrantes;
            if (rec != NULL) {
                while (rec->sig != comision->headIntegrantes) {
                    rec = rec->sig; // Encuentra el último nodo
                }
                rec->sig = nuevoNodo;         // Enlaza el nuevo nodo al final de la lista
                nuevoNodo->sig = comision->headIntegrantes; // Enlaza el nuevo nodo al nodo fantasma
            } else {
                // Si headIntegrantes es NULL, significa que es el primer nodo
                nuevoNodo->sig = nuevoNodo; // Enlaza a sí mismo
                comision->headIntegrantes = nuevoNodo; // Establece el nuevo nodo como head
            }
        }
    }
}

/*
Funciones para eliminar:
una solo eliminará a un congresista de la comision
la otra eliminará a un congresista del arreglo, por lo tanto se debe eliminar de todas las otras comisiones

igualmente que en funciones anteriores, se puede cambiar el char que recibe por el congresista entero
lo haré int para retornar 0 si hubo un fallo en la eliminacion o 1 si se eliminó bien
*/

void eliminarCongresistaDeComision(struct comision *comision, char *rutQuitado) {
    struct nodoCongresista *rec;          // Nodo actual para recorrer la lista
    struct nodoCongresista *anterior = NULL; // Nodo anterior para el manejo de la lista circular

    // Verifico si la lista está vacía o solo contiene el nodo fantasma
    if (comision == NULL || comision->headIntegrantes == NULL || comision->headIntegrantes->sig == comision->headIntegrantes) {
        return; // No existe ningún congresista en la comisión, se termina el proceso
    }

    anterior = comision->headIntegrantes; // Inicializo anterior al nodo fantasma
    rec = comision->headIntegrantes->sig;  // Inicializo rec al primer nodo real

    do {
        if (rec->datos != NULL && strcmp(rec->datos->rut, rutQuitado) == 0) {
            // Se encontró el congresista
            anterior->sig = rec->sig; // Desvincula el nodo encontrado de la lista
            free(rec->datos); // Libera la memoria del congresista (si es necesario)
            free(rec); // Libera el nodo
            return; // Salgo de la función tras eliminar
        }
        anterior = rec; // Avanzo en la lista
        rec = rec->sig;

    } while (rec != comision->headIntegrantes); // Continúo hasta volver al nodo fantasma

    // No se encontró el congresista en la lista
}

void eliminarCongresistaDeCongreso(struct congreso *congreso, char *rutQuitado) {
    struct comision **arreglo = NULL;
    int i, flag = 0;
    struct congresista *congresista;
    struct nodoCongresista *rec;
    struct nodoCongresista *recExterno, *ant;

    // Verifico que el RUT no sea NULL
    if (rutQuitado != NULL) {
        congresista = comprobarCongresistaEnCongreso(congreso, rutQuitado);

        if (congresista != NULL) {
            // Busco en el arreglo de senadores
            for (i = 0; i < MAX_SENADORES; i++) {
                if (congreso->senadores[i] != NULL && strcmp(congreso->senadores[i]->rut, rutQuitado) == 0) {
                    free(congreso->senadores[i]); // Libero memoria del congresista
                    congreso->senadores[i] = NULL;
                    flag = 1;
                }
            }
            // Si no se encontró en senadores, busco en diputados
            if (flag == 0) {
                for (i = 0; i < MAX_DIPUTADOS; i++) {
                    if (congreso->diputados[i] != NULL && strcmp(congreso->diputados[i]->rut, rutQuitado) == 0) {
                        free(congreso->diputados[i]); // Libero memoria del congresista
                        congreso->diputados[i] = NULL;
                        flag = 1;
                    }
                }
            }
            // Si aún no se ha encontrado, busco en congresistas mixtos
            if (flag == 0 && congreso->congresistasMixtos != NULL) {
                recExterno = congreso->congresistasMixtos;
                ant = NULL;
                while (recExterno != NULL && strcmp(recExterno->datos->rut, rutQuitado) != 0) {
                    ant = recExterno;
                    recExterno = recExterno->sig;
                }

                if (recExterno != NULL) {
                    if (ant == NULL) {
                        congreso->congresistasMixtos = recExterno->sig; // Eliminar el primer nodo
                    } else {
                        ant->sig = recExterno->sig; // Eliminar nodo intermedio o final
                    }
                    free(recExterno->datos); // Libero memoria del congresista
                    free(recExterno); // Libero el nodo
                    flag = 1;
                }
            }
        }

        // Ahora comienzo a borrar de las comisiones
        if (congreso->comisiones != NULL) {
            arreglo = congreso->comisiones;

            for (i = 0; i < MAX_COMISIONES; i++) {
                if (arreglo[i] != NULL) {
                    rec = arreglo[i]->headIntegrantes; // Recorro el arreglo

                    do {
                        if (comprobarCongresistaEnComision(rec, rutQuitado) == 1) {
                            // Llamo a la función para eliminar de la comisión
                            eliminarCongresistaDeComision(arreglo[i], rutQuitado);
                        }
                        rec = rec->sig;
                    } while (rec != arreglo[i]->headIntegrantes);
                }
            }
        }
    }

    if (flag == 1) {
        printf("El congresista fue eliminado\n");
    } else {
        printf("El congresista no fue encontrado\n");
    }
}

/*
Funcion para modificar a un congresista
primero tengo que elegir el arreglo en el que está o simplemente recorrer ambos arreglos
en este caso no incluí la modificacion de la ocupación
ya que al cambiar la ocupacion es mejor eliminarlo del arreglo y agregarlo al otro simplemente
*/

void modificarCongresista(struct congreso *congreso, char *rutBuscado) {
    struct congresista *congresista = comprobarCongresistaEnCongreso(congreso, rutBuscado);

    char nombre[100], especializacion[100], rut[20];
    if (congresista != NULL) {
        // Escaneo los datos nuevos
        printf("Ingrese nuevo Nombre: ");
        scanf(" %99[^\n]", nombre); // Ingrese nuevo nombre

        printf("Ingrese nueva especializacion: ");
        scanf(" %99[^\n]", especializacion); // Ingrese nueva especializacion

        printf("Ingrese nuevo rut: ");
        scanf(" %19[^\n]", rut); // Ingrese nuevo rut

        // Libero memoria previamente asignada para evitar fugas
        free(congresista->rut);
        free(congresista->nombre);
        free(congresista->especializacion);

        // Asignación de memoria para los nuevos datos
        congresista->rut = (char *)malloc(sizeof(char) * (strlen(rut) + 1));
        congresista->nombre = (char *)malloc(sizeof(char) * (strlen(nombre) + 1));
        congresista->especializacion = (char *)malloc(sizeof(char) * (strlen(especializacion) + 1));

        // Verificación de asignación de memoria
        if (congresista->rut == NULL || congresista->nombre == NULL || congresista->especializacion == NULL) {
            printf("Error al asignar memoria para los datos del congresista.\n");
            return; // Salgo si hubo error en la asignación
        }

        // Copiar los nuevos datos
        strcpy(congresista->rut, rut);
        strcpy(congresista->nombre, nombre);
        strcpy(congresista->especializacion, especializacion);

        printf("Fue modificado correctamente\n");
        return;
    }
    printf("Hubo un problema al modificar al congresista\n");
}

void mostrarCongresista(struct congreso *congreso, char *rutBuscado) {
    struct congresista *congresistaBuscado = NULL;

    // Busco el congresista
    congresistaBuscado = comprobarCongresistaEnCongreso(congreso, rutBuscado);
    if (congresistaBuscado == NULL) {
        printf("No existe el congresista en el sistema\n");
        return;
    }

    // Muestra la información del congresista
    printf("Nombre: %s\n", congresistaBuscado->nombre);
    printf("RUT: %s\n", congresistaBuscado->rut);
    printf("Ocupacion: %s\n", congresistaBuscado->ocupacion);
    printf("Especializacion: %s\n\n", congresistaBuscado->especializacion);
}

/*
funcion listar congresista:
esta funcion es un poco redundante, pues llama al mostrarCongresista y esta lo busca de nuevo pero cumple con el requisito de listar
*/
void listarCongresistas(struct congreso *congreso) {
    struct congresista **arregloDiputados = congreso->diputados;
    struct congresista **arregloSenadores = congreso->senadores;
    struct nodoCongresista *rec;
    int i, flagDiputados = 0, flagSenadores = 0, flagExternos = 0;

    // Verifica que el congreso no sea NULL
    if (congreso == NULL) {
        return;
    }

    printf("Diputados en sistema:\n");
    if (arregloDiputados != NULL) {
        for (i = 0; i < MAX_DIPUTADOS; i++) {
            if (arregloDiputados[i] != NULL) {
                printf("Nombre: %s\n", arregloDiputados[i]->nombre);
                printf("RUT: %s\n\n", arregloDiputados[i]->rut);
                flagDiputados = 1;
            }
        }
    }
    printf("\n\n");

    if (arregloSenadores != NULL) {
        printf("Senadores en sistema:\n");
        for (i = 0; i < MAX_SENADORES; i++) {
            if (arregloSenadores[i] != NULL) {
                printf("Nombre: %s\n", arregloSenadores[i]->nombre);
                printf("RUT: %s\n\n", arregloSenadores[i]->rut);
                flagSenadores = 1;
            }
        }
    }
    printf("\n\n");

    if (congreso->congresistasMixtos != NULL) {
        printf("Externos en sistema:\n");
        rec = congreso->congresistasMixtos;
        while (rec != NULL) {
            if (rec->datos != NULL) {
                printf("Nombre: %s\n", rec->datos->nombre);
                printf("RUT: %s\n\n", rec->datos->rut);
                flagExternos = 1;
            }
            rec = rec->sig; // Se asume que esto es correcto, aunque normalmente la lista enlazada debería tener un nodo final
        }
    }

    if (flagDiputados == 0 && flagSenadores == 0 && flagExternos == 0) {
        printf("El sistema no tiene congresistas\n");
    }
}

/*TODO: FUNCIONES DE COMISIONES------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/
/*TODO------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/

struct nodoComision* crearNodoComision(struct comision *datos) {
    // Asignar memoria para el nuevo nodo
    struct nodoComision *nuevoNodo = (struct nodoComision *)malloc(sizeof(struct nodoComision));
    if (nuevoNodo == NULL) {
        // Manejo de error en caso de que malloc falle
        return NULL;
    }

    // Inicializar el nodo
    nuevoNodo->datos = datos;
    nuevoNodo->sig = NULL;

    return nuevoNodo; // Retornar el nuevo nodo
}

struct comision *buscarComision(struct congreso *congreso, char *nombre) {
    int i;
    struct nodoComision *rec = NULL;
    struct comision *buscado = NULL;

    for (i = 0; i < MAX_COMISIONES; i++) {
        if (congreso->comisiones[i] != NULL && strcmp(congreso->comisiones[i]->nombre, nombre) == 0) {
            buscado = congreso->comisiones[i];
            return buscado; // Retorna la comisión si se encuentra
        }
    }

    rec = congreso->comisionesMixtas;
    while (rec != NULL) {
        if (strcmp(rec->datos->nombre, nombre) == 0) {
            buscado = rec->datos;
            return buscado; // Retorna la comisión si se encuentra
        }
        rec = rec->sig; // Avanza al siguiente nodo
    }

    return buscado; // Retorna NULL si no se encuentra
}

struct comision *crearComision(struct congreso *congreso) {
    struct comision *nuevaComision = (struct comision*)malloc(sizeof(struct comision));
    if (nuevaComision == NULL) {
        printf("Error al asignar memoria\n");
        return NULL;
    }

    char nombre[100], tipo[50], descripcion[256];

    printf("Ingresa el nombre de la comision:\n");
    scanf("%99[^\n]", nombre);
    limpiarBuffer();

    if (buscarComision(congreso, nombre) != NULL) {
        printf("La comision ya existe en el sistema\n");
        free(nuevaComision);
        return NULL;
    }

    printf("Ingresa descripcion de la comision:\n");
    scanf("%255[^\n]", descripcion);
    limpiarBuffer();

    printf("Ingresa el tipo de la comision (senadores/diputados/otros):\n");
    scanf("%49[^\n]", tipo);
    limpiarBuffer();

    convertirMinusculas(tipo);

    // Asignar memoria para los campos de la nueva comisión
    nuevaComision->nombre = (char*)malloc(sizeof(char) * (strlen(nombre) + 1));
    nuevaComision->descripcion = (char*)malloc(sizeof(char) * (strlen(descripcion) + 1));
    nuevaComision->tipo = (char*)malloc(sizeof(char) * (strlen(tipo) + 1));

    if (nuevaComision->nombre == NULL || nuevaComision->descripcion == NULL || nuevaComision->tipo == NULL) {
        printf("Error al asignar memoria de uno de los campos\n");
        free(nuevaComision->nombre);
        free(nuevaComision->descripcion);
        free(nuevaComision->tipo);
        free(nuevaComision);
        return NULL;
    }

    // Copiar los datos a la nueva comisión
    strcpy(nuevaComision->nombre, nombre);
    strcpy(nuevaComision->descripcion, descripcion);
    strcpy(nuevaComision->tipo, tipo);

    // Crear el nodo fantasma
    struct nodoCongresista *fantasma = (struct nodoCongresista *)malloc(sizeof(struct nodoCongresista));
    if (fantasma == NULL) {
        printf("Error al crear el nodo fantasma\n");
        free(nuevaComision->nombre);
        free(nuevaComision->descripcion);
        free(nuevaComision->tipo);
        free(nuevaComision);
        return NULL;
    }
    fantasma->datos = NULL;
    fantasma->sig = fantasma; // Apunta a sí mismo, indicando que es un nodo fantasma
    nuevaComision->headIntegrantes = fantasma;

    return nuevaComision; // Retorna la nueva comisión creada
}


void agregarComision(struct congreso *congreso) {
    struct comision *nuevaComision = crearComision(congreso);
    struct nodoComision *nuevoNodo = NULL;

    if (nuevaComision == NULL) {
        printf("Error al crear la nueva comision\n");
        return;
    }

    if (strcmp(nuevaComision->tipo, "senadores") == 0 || strcmp(nuevaComision->tipo, "diputados") == 0) {
        // Intentar agregar al arreglo de comisiones
        for (int i = 0; i < MAX_COMISIONES; i++) {
            if (congreso->comisiones[i] == NULL) {
                congreso->comisiones[i] = nuevaComision;
                printf("Comision agregada al arreglo de comisiones\n");
                return;
            }
        }
        // Si se alcanza el límite del arreglo
        printf("Error al asignar en el arreglo: se ha alcanzado el máximo de comisiones\n");
    } else {
        // Agregar como comisión mixta
        nuevoNodo = crearNodoComision(nuevaComision);
        if (nuevoNodo == NULL) {
            printf("Error al asignar memoria al nodo\n");
            return;
        }

        nuevoNodo->sig = congreso->comisionesMixtas;
        congreso->comisionesMixtas = nuevoNodo;
        printf("Comision agregada como mixta\n");
    }
}

void eliminarComision(struct congreso *congreso, char *nombre) {
    struct comision *buscada = buscarComision(congreso, nombre);
    struct nodoComision *rec, *ant;

    if (buscada == NULL) {
        printf("Error al eliminar la comision: no existe en el sistema\n");
        return;
    }

    // Intentar eliminar del arreglo de comisiones
    for (int i = 0; i < MAX_COMISIONES; i++) {
        if (congreso->comisiones[i] == buscada) {
            free(congreso->comisiones[i]->nombre);
            free(congreso->comisiones[i]->descripcion);
            free(congreso->comisiones[i]->tipo);
            free(congreso->comisiones[i]->headIntegrantes);
            free(congreso->comisiones[i]); // Liberar la estructura de la comisión
            congreso->comisiones[i] = NULL; // Marcar como vacío
            printf("Comision eliminada correctamente del arreglo\n");
            return;
        }
    }

    // Si no está en el arreglo, buscar en la lista de comisiones mixtas
    rec = congreso->comisionesMixtas;
    ant = NULL;

    while (rec != NULL) {
        if (rec->datos == buscada) {
            // Caso del primer nodo
            if (ant == NULL) {
                congreso->comisionesMixtas = rec->sig; // Mover la cabeza de la lista
            } else {
                ant->sig = rec->sig; // Desvincular el nodo
            }
            // Liberar memoria de la comisión
            free(rec->datos->nombre);
            free(rec->datos->descripcion);
            free(rec->datos->tipo);
            free(rec->datos); // Liberar la estructura de la comisión
            free(rec); // Liberar el nodo de la lista
            printf("Comisión eliminada correctamente de la lista\n");
            return;
        }
        ant = rec;
        rec = rec->sig;
    }

    printf("Error al eliminar la comision: no se encontró en la lista de comisiones mixtas\n");
}

void modificarComision(struct congreso *congreso, char *nombre) {
    struct congresista *nuevoCongresista = NULL;
    struct comision *comisionAModificar = buscarComision(congreso, nombre);
    int opcion, subOpcion;
    char nuevoNombre[100], nuevoTipo[50], nuevaDescripcion[256], rut[20];

    if (comisionAModificar == NULL) {
        printf("No existe una comision con el nombre especificado.\n");
        return;
    }

    printf("Seleccione el campo que desea modificar:\n");
    printf("1. Nombre\n");
    printf("2. Tipo\n");
    printf("3. Descripcion\n");
    printf("4. Modificar Miembros\n");
    printf("Ingrese su opción: ");
    scanf("%d", &opcion);
    limpiarBuffer();

    switch (opcion) {
        case 1: // Modificar nombre
            printf("Ingrese el nuevo nombre de la comision:\n");
            scanf("%99[^\n]", nuevoNombre);
            limpiarBuffer();

            if (buscarComision(congreso, nuevoNombre) != NULL) {
                printf("El nombre ya está en uso. Modificación cancelada.\n");
                return;
            }

            free(comisionAModificar->nombre);  // Liberar memoria del nombre anterior
            comisionAModificar->nombre = (char*)malloc(strlen(nuevoNombre) + 1);
            if (comisionAModificar->nombre == NULL) {
                printf("Error al asignar memoria para el nuevo nombre.\n");
                return;
            }
            strcpy(comisionAModificar->nombre, nuevoNombre);
            printf("Nombre de la comisión actualizado correctamente.\n");
            break;

        case 2: // Modificar tipo
            printf("Ingrese el nuevo tipo de la comisión (senadores/diputados/otros):\n");
            scanf("%49[^\n]", nuevoTipo);
            limpiarBuffer();

            free(comisionAModificar->tipo);  // Liberar memoria del tipo anterior
            comisionAModificar->tipo = (char*)malloc(strlen(nuevoTipo) + 1);
            if (comisionAModificar->tipo == NULL) {
                printf("Error al asignar memoria para el nuevo tipo.\n");
                return;
            }
            strcpy(comisionAModificar->tipo, nuevoTipo);
            printf("Tipo de la comisión actualizado correctamente.\n");
            break;

        case 3: // Modificar descripción
            printf("Ingrese la nueva descripción de la comisión:\n");
            scanf("%255[^\n]", nuevaDescripcion);
            limpiarBuffer();

            free(comisionAModificar->descripcion);  // Liberar memoria de la descripción anterior
            comisionAModificar->descripcion = (char*)malloc(strlen(nuevaDescripcion) + 1);
            if (comisionAModificar->descripcion == NULL) {
                printf("Error al asignar memoria para la nueva descripción.\n");
                return;
            }
            strcpy(comisionAModificar->descripcion, nuevaDescripcion);
            printf("Descripción de la comisión actualizada correctamente.\n");
            break;

        case 4: // Modificar miembros
            printf("Menú de modificar congresistas en la comisión\n");
            printf("Elija una opción:\n");
            printf("1. Agregar Congresista a la comisión\n");
            printf("2. Eliminar Congresista de la comisión\n");
            scanf("%d", &subOpcion);
            limpiarBuffer();

            switch (subOpcion) {
                case 1: // Agregar congresista
                    printf("Ingrese el RUT del congresista a agregar\n");
                    scanf("%19s", rut);
                    limpiarBuffer();
                    nuevoCongresista = comprobarCongresistaEnCongreso(congreso, rut);
                    if (nuevoCongresista != NULL) {
                        if (comprobarCongresistaEnComision(comisionAModificar->headIntegrantes, rut) == 1) {
                            printf("Este congresista ya se encuentra en la lista\n");
                        } else {
                            agregarCongresistaEnComision(comisionAModificar, rut, nuevoCongresista);
                            printf("Congresista agregado correctamente\n");
                        }
                    } else {
                        printf("Congresista no encontrado en el sistema\n");
                    }
                    break;

                case 2: // Eliminar congresista
                    printf("Ingrese el RUT del congresista a eliminar\n");
                    scanf("%19s", rut);
                    limpiarBuffer();
                    if (comprobarCongresistaEnComision(comisionAModificar->headIntegrantes, rut) == 1) {
                        eliminarCongresistaDeComision(comisionAModificar, rut);
                        printf("Congresista eliminado correctamente\n");
                    } else {
                        printf("No se encontró al congresista en la comisión\n");
                    }
                    break;

                default:
                    printf("Opción no válida\n");
                    break;
            }
            break;

        default:
            printf("Opción no válida.\n");
            break;
    }
}

void mostrarComisionPorNombre(struct congreso *congreso, char *nombre) {
    struct comision *buscada = buscarComision(congreso, nombre);
    struct nodoCongresista *rec;

    if (buscada == NULL) {
        printf("No existe una comision con este nombre\n");
        return;
    }

    printf("Comision encontrada:\n\n");
    printf("Nombre: %s\n", buscada->nombre);
    printf("Descripcion: %s\n", buscada->descripcion);
    printf("Tipo: %s\n", buscada->tipo);

    rec = buscada->headIntegrantes;
    if (rec == NULL || rec->sig == rec) { // Chequeo si no hay integrantes
        printf("No hay integrantes en la comision\n");
    } else {
        printf("Integrantes de la comision:\n\n");
        rec = rec->sig; // Saltar el nodo fantasma
        do {
            if (rec->datos != NULL) {
                printf("- Nombre: %s\n", rec->datos->nombre);
                printf("- Ocupacion: %s\n\n", rec->datos->ocupacion);
            }
            rec = rec->sig;
        } while (rec != buscada->headIntegrantes); // Termina cuando regresa al nodo fantasma
    }
}

void listarComisiones(struct congreso *congreso) {
    int i;
    struct nodoComision *rec;

    // Verificar si el congreso es NULL
    if (congreso == NULL) {
        printf("El congreso no está inicializado.\n");
        return;
    }

    printf("Comisiones de senadores y diputados:\n");
    for (i = 0; i < MAX_COMISIONES; i++) {
        if (congreso->comisiones[i] != NULL) {
            printf("Nombre: %s\n", congreso->comisiones[i]->nombre);
            printf("Descripcion: %s\n", congreso->comisiones[i]->descripcion);
            printf("Tipo: %s\n", congreso->comisiones[i]->tipo);
            printf("\n");
        }
    }

    printf("Comisiones mixtas:\n");
    rec = congreso->comisionesMixtas;
    while (rec != NULL) {
        if (rec->datos != NULL) { // Verificar que los datos no sean NULL
            printf("Nombre: %s\n", rec->datos->nombre);
            printf("Descripcion: %s\n", rec->datos->descripcion);
            printf("Tipo: %s\n", rec->datos->tipo);
            printf("\n");
        }
        rec = rec->sig;
    }
}

/*TODO: FUNCIONES DE ARTICULOS------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/
/*TODO------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/

void copiarCambioATexto(struct nodoArticulo *articulos) {
    int seccionBuscada;
    struct nodoArticulo *rec = articulos;

    // Pedir al usuario la sección a modificar
    printf("Ingrese la sección del artículo a modificar: ");
    scanf("%d", &seccionBuscada);

    // Buscar el artículo con la sección correspondiente
    while (rec != NULL) {
        if (rec->datos != NULL && rec->datos->seccion == seccionBuscada) {
            // Verificar que "cambio" no sea NULL antes de copiar
            if (rec->datos->cambio != NULL) {
                // Liberar memoria previa si existe
                if (rec->datos->texto != NULL) {
                    free(rec->datos->texto);
                }

                // Asignar memoria para "texto" y copiar el contenido de "cambio"
                rec->datos->texto = malloc(strlen(rec->datos->cambio) + 1); // +1 para el terminador nulo
                if (rec->datos->texto != NULL) {
                    strcpy(rec->datos->texto, rec->datos->cambio);
                    printf("El texto de la sección %d ha sido actualizado correctamente.\n", seccionBuscada);
                } else {
                    printf("Error al asignar memoria para el texto.\n");
                }
            } else {
                printf("No hay contenido en 'cambio' para copiar.\n");
            }
            return; // Salir de la función después de la modificación
        }
        rec = rec->sig; // Avanzar al siguiente nodo
    }
    printf("No se encontró un artículo con la sección %d.\n", seccionBuscada);
}

struct nodoArticulo *crearNodoArticulo(struct articulo *datos) {
    // Verificar que los datos recibidos no sean NULL
    if (datos == NULL) {
        return NULL; // Retornar NULL si no hay datos
    }

    // Asignar memoria para el nuevo nodo
    struct nodoArticulo *nodo = (struct nodoArticulo *)malloc(sizeof(struct nodoArticulo));
    if (nodo == NULL) {
        // Manejo de error en caso de que malloc falle
        return NULL;
    }

    // Asignar los datos al nuevo nodo
    nodo->datos = datos;

    // Inicializar los punteros siguiente y anterior
    nodo->sig = NULL;
    nodo->ant = NULL;

    return nodo; // Retornar el nuevo nodo creado
}

/*

comprobar que exista articulo, lo haré de manera que retorne 0 si NO existe el articulo, o que retorne 1 si existe
la idea es que la seccion sea el buscado, por lo tanto las otras funciones que la llamen deben ingresar la seccion
aunque esto puede estar sujeto a cambios si se desea, quizas recibir el nodo entero para comodidad

*/
int comprobarArticulo(struct nodoArticulo *head, int buscado) {
    // Verificar si la lista está vacía
    if (head == NULL) {
        return 0; // La lista está vacía, no se encuentra el artículo
    }

    struct nodoArticulo *rec = head; // Inicializar el nodo de recorrido

    // Recorrer la lista en busca del artículo
    while (rec != NULL) {
        if (rec->datos != NULL && rec->datos->seccion == buscado) {
            return 1; // Se encontró el artículo en la lista
        }
        rec = rec->sig; // Avanzar al siguiente nodo
    }

    return 0; // No se encontró el artículo en la lista
}

/*
Agregar el nuevo articulo, llamará a 2 funciones, la de crear el nodo y la de crear el nuevo articulo
la idea es que primero se creen los datos del articulo y luego se cree el nodo, la funcion crear nodo articulo
recibirá los datos de la función crear articulo

PD: todos los prints serán eliminados posterior a la creacion del main
*/

struct articulo *crearArticulo(struct nodoArticulo *lista) {
    struct articulo *nuevoArticulo = (struct articulo *)malloc(sizeof(struct articulo));
    char nombre[100], texto[4096], cambio[4096];
    int seccion;

    // Verificar si la asignación de memoria fue exitosa
    if (nuevoArticulo == NULL) {
        printf("Error al asignar memoria para el nuevo artículo.\n");
        return NULL;
    }

    printf("Ingresa el número de sección del artículo: ");
    scanf("%d", &seccion); // Recibir el número de sección
    nuevoArticulo->seccion = seccion;
    nuevoArticulo->voto = NULL; // Inicializar voto como NULL

    // Recibir el nombre del artículo
    printf("Ingrese el nombre: ");
    scanf(" %[^\n]", nombre);
    limpiarBuffer(); // Asegurarse de limpiar el buffer

    // Recibir la descripción del artículo
    printf("Ingrese la descripción del artículo: ");
    scanf(" %[^\n]", texto);
    limpiarBuffer();

    // Recibir los cambios del artículo
    printf("Ingrese los cambios: ");
    scanf(" %[^\n]", cambio);
    limpiarBuffer();

    // Validar la longitud de las cadenas
    if (strlen(nombre) > 100 || strlen(texto) > 256 || strlen(cambio) > 256) {
        printf("Uno de los valores ingresados es demasiado largo.\n");
        free(nuevoArticulo); // Liberar memoria asignada a nuevoArticulo
        return NULL;
    }

    // Asignar memoria para los campos de texto
    nuevoArticulo->nombre = (char *)malloc(strlen(nombre) + 1);
    nuevoArticulo->texto = (char *)malloc(strlen(texto) + 1);
    nuevoArticulo->cambio = (char *)malloc(strlen(cambio) + 1);

    // Verificar si la asignación de memoria fue exitosa
    if (nuevoArticulo->nombre == NULL || nuevoArticulo->texto == NULL || nuevoArticulo->cambio == NULL) {
        printf("Error al asignar memoria para uno de los campos del artículo.\n");
        free(nuevoArticulo->nombre); // Liberar memoria en caso de error
        free(nuevoArticulo->texto);
        free(nuevoArticulo->cambio);
        free(nuevoArticulo);
        return NULL; // Retornar NULL en caso de error
    }

    // Copiar los valores a los campos del artículo
    strcpy(nuevoArticulo->nombre, nombre);
    strcpy(nuevoArticulo->texto, texto);
    strcpy(nuevoArticulo->cambio, cambio);

    return nuevoArticulo; // Retornar el nuevo artículo creado
}

void agregarArticulo(struct congreso *congreso, struct nodoArticulo **lista) {
    printf("Depuración: Valor de *lista al entrar en agregarArticulo: %p\n", (void *)*lista);
    struct nodoArticulo *NuevoArticulo;
    struct nodoArticulo *rec;
    struct articulo *datos;

    datos = crearArticulo(*lista);
    if (datos == NULL) {
        printf("Error al asignar datos\n");
        return;
    }

    NuevoArticulo = crearNodoArticulo(datos);
    if (NuevoArticulo != NULL) {
        if (*lista == NULL) {
            (*lista) = NuevoArticulo; // La lista estaba vacía, se agrega sin problemas
            return;
        } else {
            if (comprobarArticulo(*lista, datos->seccion) == 0) {
                rec = *lista;
                printf("se cae antes de llegar a la ultima pos");
                // Llego a la última posición de la lista
                while (rec->sig != NULL) {
                    rec = rec->sig;
                }
                rec->sig = NuevoArticulo; // Llego al final y lo agrego
                NuevoArticulo->ant = rec;
                printf("Agregado correctamente.\n");
                return; // Agregado correctamente
            } else {
                printf("El artículo ya existe en la lista.\n");
            }
        }
    }
    // No se cumplen los requisitos, no se agrega
}

/*
El eliminar articulo tambien será int para que en la consola sea mas facil poner algo como: articulo eliminado, o error
si devuelve 1, ha sido un exito la eliminacion, si retorna 0, no se encontró en la lista
la funcion recibe la lista de articulos, la idea es que se seleccione la ley y a partir de ahi se elimine el articulo
*/


int eliminarArticulo(struct nodoArticulo **lista, int seccionEliminada) {
    struct nodoArticulo *rec;

    // Verificamos si la lista existe
    if (*lista != NULL) {
        rec = *lista;

        // Caso 1: El artículo está en la primera posición
        if (seccionEliminada == rec->datos->seccion) {
            *lista = rec->sig; // Mover el encabezado de la lista
            if (*lista != NULL) {
                (*lista)->ant = NULL; // Actualizamos el puntero anterior del nuevo encabezado, si existe
            }
            free(rec->datos); // Liberar memoria del artículo
            free(rec); // Liberar el nodo
            return 1; // Se ha encontrado y eliminado el artículo
        }

        // Caso 2: Se encuentra en cualquier otra posición de la lista
        while (rec->sig != NULL) {
            if (seccionEliminada == rec->sig->datos->seccion) {
                struct nodoArticulo *nodoAEliminar = rec->sig; // Nodo a eliminar
                rec->sig = nodoAEliminar->sig; // Saltar el nodo a eliminar
                if (nodoAEliminar->sig != NULL) {
                    nodoAEliminar->sig->ant = rec; // Actualizar el puntero anterior del siguiente nodo
                }
                free(nodoAEliminar->datos); // Liberar memoria del artículo
                free(nodoAEliminar); // Liberar el nodo
                return 1; // Se ha encontrado y eliminado el artículo
            }
            rec = rec->sig; // Avanzar al siguiente nodo
        }
    }
    return 0; // No se encontró el artículo
}

//return 1: modificado de forma correcta return 0: no se pudo modificar

int modificarArticulo(struct nodoArticulo *articulos, int seccionModificada) {
    struct nodoArticulo *rec;
    struct articulo *articuloBuscado = NULL;
    char nombre[100];
    char texto[4096];
    char cambio[4096];

    // Se escanean los datos nuevos
    printf("Ingrese el nuevo nombre:\n");
    scanf(" %[^\n]", nombre); // Espacio antes de % para ignorar nuevos lineas
    printf("Ingrese el nuevo texto:\n");
    scanf(" %[^\n]", texto);
    printf("Ingrese los nuevos cambios:\n");
    scanf(" %[^\n]", cambio);

    // Validar longitud de los nuevos textos
    if (strlen(texto) > 4096 || strlen(cambio) > 4096) {
        printf("Uno de los valores ingresados es muy largo.\n");
        return 0;
    }

    // Verificamos que la lista no sea NULL
    if (articulos != NULL) {
        rec = articulos;

        // Recorro y busco el artículo específico
        while (rec != NULL) {
            // Si se encuentra el artículo a modificar
            if (rec->datos->seccion == seccionModificada) {
                articuloBuscado = rec->datos; // Se copia la info del artículo encontrado

                // Asignar nueva memoria para los campos
                free(articuloBuscado->nombre); // Liberar memoria existente
                articuloBuscado->nombre = (char *)malloc(sizeof(char) * (strlen(nombre) + 1));
                if (articuloBuscado->nombre == NULL) {
                    printf("Error al asignar memoria para el nombre.\n");
                    return 0;
                }

                // Asignar memoria para texto y cambio
                free(articuloBuscado->texto); // Liberar memoria existente
                articuloBuscado->texto = (char *)malloc(sizeof(char) * 4096);
                if (articuloBuscado->texto == NULL) {
                    printf("Error al asignar memoria para el texto.\n");
                    return 0;
                }

                free(articuloBuscado->cambio); // Liberar memoria existente
                articuloBuscado->cambio = (char *)malloc(sizeof(char) * 4096);
                if (articuloBuscado->cambio == NULL) {
                    printf("Error al asignar memoria para el cambio.\n");
                    return 0;
                }

                // Copio los datos
                strcpy(articuloBuscado->nombre, nombre);
                strcpy(articuloBuscado->texto, texto);
                strcpy(articuloBuscado->cambio, cambio);

                return 1; // Modificado correctamente
            }
            rec = rec->sig;
        }
    }
    return 0; // No se logra modificar
}

/*TODO: FUNCIONES CON LOS SWITCH (MENUS)----------------------------------------------------------------------------------------------------------------------------------------------------*/
/*TODO:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/

void funcionSwitch(char opcion, struct congreso *congreso, void (*submenu)(struct congreso *)) {
    switch (opcion) {
        case 'A':
            printf("Seleccionaste la opcion A. Accediendo al menu...\n");
        submenu(congreso);
        break;
        case 'B':
            printf("Seleccionaste la opcion B. Accediendo al menu...\n");
        submenu(congreso);
        break;
        case 'C':
            printf("Seleccionaste la opcion C. Accediendo al menu...\n");
        submenu(congreso);
        break;
        case 'D':
            printf("Seleccionaste la opcion D. Accediendo al menu...\n");
        submenu(congreso);
        break;
        case 'E':
            printf("Seleccionaste la opcion E. Cerrando el menu.\n");
        break;
        default:
            printf("Opcion invalida, por favor intente otra vez.\n");
        break;
    }
}

void menuProyectosLey(struct congreso *congreso) {
    char opcion[2];
    int id, fase;
    while (1) {
        printf("Menu Proyectos de Ley.\n"
            "Opcion A: Agregar nuevo Proyecto de Ley\n"
            "Opcion B: Borrar Proyecto de Ley\n"
            "Opcion C: Mostrar Proyecto de Ley\n"
            "Opcion D: Modificar Proyecto de Ley\n"
            "Opcion E: Listar Proyectos de Ley\n"
            "Opcion F: Mostrar Proyectos en Orden de Urgencia\n"
            "Opcion G: Mostrar Proyectos por fase\n"
            "Opcion H: Volver al menu principal\n");

        scanf("%1s", opcion);
        limpiarBuffer();

        // Convierte la opción a mayúscula si está en minúscula
        opcion[0] = (opcion[0] >= 'a' && opcion[0] <= 'z') ? opcion[0] - ('a' - 'A') : opcion[0];

        switch (opcion[0]) {
            case 'A':
                printf("Funcion: Agregar nuevo Proyecto de Ley\n");
                agregarNodoProyectoLey(congreso);
                break;
            case 'B':
                printf("Funcion: Borrar Proyecto de Ley\n");
                id = leerEnteroConLimite("Ingrese la ID del Proyecto que quiere borrar", 10000000, 99999999);
                borrarProyectoLey(congreso, id);
                break;
            case 'C':
                printf("Funcion: Mostrar Proyecto de Ley\n");
                id = leerEnteroConLimite("Ingrese la ID del Proyecto que quiere buscar", 10000000, 99999999);
                buscarYMostrarProyectoLey(congreso, id);
                break;
            case 'D':
                printf("Funcion: Modificar Proyecto de Ley\n");
                id = leerEnteroConLimite("Ingrese la ID del Proyecto que quiere modificar", 10000000, 99999999);
                modificarProyectoLey(congreso, id);
                break;
            case 'E':
                printf("Funcion: Listar Proyectos de Ley\n");
                imprimirProyectosLey(congreso);
                break;
            case 'F':
                printf("Funcion: Mostrar Proyectos en Orden de Urgencia\n");
                mostrarProyectosOrdenDeUrgencia(congreso);
                break;
            case 'G':
                printf("Funcion: Mostrar Proyectos por fase\n");
                fase = leerEnteroConLimite("Ingrese la fase de los proyectos que quiere mostrar", 1, 8);
                mostrarLeyesPorFase(congreso->raiz, fase);
                break; // Se añadió break aquí para evitar caer en el caso 'H'
            case 'H':
                return; // Salir del menú
            default:
                printf("Opcion invalida, por favor intente otra vez.\n");
                break;
        }
    }
}

void menuCongresistas(struct congreso *congreso) {
    char opcion[2];
    char rut[20];

    while (1) {
        printf("Menu Congresistas.\n"
            "Opcion A: Agregar Congresista\n"
            "Opcion B: Borrar Congresista\n"
            "Opcion C: Buscar Congresista\n"
            "Opcion D: Modificar Congresista\n"
            "Opcion E: Listar Congresistas\n"
            "Opcion F: Volver al menu principal\n");

        scanf("%1s", opcion);
        limpiarBuffer();

        // Convierte la opción a mayúscula si está en minúscula
        opcion[0] = (opcion[0] >= 'a' && opcion[0] <= 'z') ? opcion[0] - ('a' - 'A') : opcion[0];

        switch (opcion[0]) {
            case 'A':
                printf("Funcion: Agregar Congresista\n");
                agregarCongresistaEnCongreso(congreso);
                break; // Se añadió break aquí
            case 'B':
                printf("Funcion: Borrar Congresista\n Ingrese el rut del congresista a eliminar: ");
                scanf("%19[^\n]", rut);
                limpiarBuffer();
                eliminarCongresistaDeCongreso(congreso, rut);
                break; // Se añadió break aquí
            case 'C':
                printf("Funcion: Buscar Congresista\n");
                printf("Ingrese el rut del congresista a buscar: ");
                scanf("%19[^\n]", rut);
                limpiarBuffer();
                mostrarCongresista(congreso, rut);
                break; // Se añadió break aquí
            case 'D':
                printf("Funcion: Modificar Congresista\n");
                printf("Ingrese el rut del congresista a modificar:\n");
                scanf(" %19[^\n]", rut);
                limpiarBuffer();
                modificarCongresista(congreso, rut);
                break; // Se añadió break aquí
            case 'E':
                printf("Funcion: Listar Congresistas\n");
                listarCongresistas(congreso);
                break; // Se añadió break aquí
            case 'F':
                return; // Salir del menú
            default:
                printf("Opcion invalida, por favor intente otra vez.\n");
                break; // Se añadió break aquí
        }
    }
}

void gestionarVotacionArticulo(struct congreso *congreso, struct nodoArticulo *articulos) {
    int seccionBuscada;
    char opcionVoto;
    char rut[20]; // RUT en formato 12.345.678-K
    struct nodoArticulo *rec = articulos;

    // Pedir al usuario la sección del artículo para agregar la votación
    printf("Ingrese la sección del artículo para añadir una votación: ");
    scanf("%d", &seccionBuscada);
    limpiarBuffer();

    // Buscar el artículo con la sección especificada
    while (rec != NULL) {
        if (rec->datos != NULL && rec->datos->seccion == seccionBuscada) {
            // Inicializar la votación si no existe
            if (rec->datos->voto == NULL) {
                rec->datos->voto = (struct votacion *)malloc(sizeof(struct votacion));
                if (rec->datos->voto == NULL) {
                    printf("Error al asignar memoria para la votación.\n");
                    return;
                }
                rec->datos->voto->favor = NULL;
                rec->datos->voto->contra = NULL;
            }

            // Preguntar al usuario si desea añadir un congresista a favor o en contra
            printf("¿Desea añadir un voto (F)avor o (C)ontra? ");
            scanf(" %c", &opcionVoto);
            limpiarBuffer();

            if (opcionVoto == 'F' || opcionVoto == 'f' || opcionVoto == 'C' || opcionVoto == 'c') {
                printf("Ingrese el RUT del congresista (formato 12.345.678-K): ");
                scanf("%19[^\n]", rut);
                limpiarBuffer();

                // Buscar al congresista en el congreso usando el RUT
                struct congresista *congresista = comprobarCongresistaEnCongreso(congreso, rut);
                if (congresista == NULL) {
                    printf("Error: Congresista con RUT %s no encontrado.\n", rut);
                    return;
                }

                // Crear un nuevo nodo de congresista para añadirlo a la votación
                struct nodoCongresista *nuevoNodo = (struct nodoCongresista *)malloc(sizeof(struct nodoCongresista));
                if (nuevoNodo == NULL) {
                    printf("Error al asignar memoria para el nodo de congresista.\n");
                    return;
                }
                nuevoNodo->datos = congresista;
                nuevoNodo->sig = NULL;

                // Añadir el nodo a la lista correspondiente (favor o contra)
                if (opcionVoto == 'F' || opcionVoto == 'f') {
                    struct nodoCongresista *actual = rec->datos->voto->favor;
                    if (actual == NULL) {
                        rec->datos->voto->favor = nuevoNodo;
                    } else {
                        while (actual->sig != NULL) {
                            actual = actual->sig;
                        }
                        actual->sig = nuevoNodo;
                    }
                    printf("Congresista añadido a la lista de votos a favor.\n");
                } else {
                    struct nodoCongresista *actual = rec->datos->voto->contra;
                    if (actual == NULL) {
                        rec->datos->voto->contra = nuevoNodo;
                    } else {
                        while (actual->sig != NULL) {
                            actual = actual->sig;
                        }
                        actual->sig = nuevoNodo;
                    }
                    printf("Congresista añadido a la lista de votos en contra.\n");
                }
            } else {
                printf("Opción de voto inválida.\n");
            }
            return; // Salir de la función después de procesar la votación
        }
        rec = rec->sig; // Avanzar al siguiente nodo
    }
    printf("No se encontró un artículo con la sección %d.\n", seccionBuscada);
}

void mostrarVotacionArticulos(struct proyectoLey *ley) {
    struct nodoArticulo *articuloActual = ley->articulo;
    int contadorFavor, contadorContra;

    while (articuloActual != NULL) {
        struct articulo *art = articuloActual->datos;
        printf("\nSección del Artículo: %d\n", art->seccion);
        printf("Texto del Artículo: %s\n", art->texto);
        printf("Cambios Pendientes: %s\n", art->cambio);

        // Mostrar votación a favor
        contadorFavor = 0;
        printf("Votos a Favor:\n");
        if (art->voto != NULL && art->voto->favor != NULL) {
            struct nodoCongresista *votante = art->voto->favor;
            while (votante != NULL) {
                printf("  - RUT: %s\n", votante->datos->rut);
                contadorFavor++;
                votante = votante->sig;
            }
        } else {
            printf("  No hay votos a favor.\n");
        }

        // Mostrar votación en contra
        contadorContra = 0;
        printf("Votos en Contra:\n");
        if (art->voto != NULL && art->voto->contra != NULL) {
            struct nodoCongresista *votante = art->voto->contra;
            while (votante != NULL) {
                printf("  - RUT: %s\n", votante->datos->rut);
                contadorContra++;
                votante = votante->sig;
            }
        } else {
            printf("  No hay votos en contra.\n");
        }

        // Mostrar el recuento total de votos
        printf("Total Votos a Favor: %d\n", contadorFavor);
        printf("Total Votos en Contra: %d\n", contadorContra);

        articuloActual = articuloActual->sig; // Avanzar al siguiente artículo
    }
}

void menuArticulos(struct congreso *congreso, struct proyectoLey *ley) {
    char opcion;
    int seccionEliminar;
    int seccionModificar;

    do {
        printf("Seleccione la acción sobre los artículos:\n");
        printf("A. Agregar Artículo\n");
        printf("B. Modificar Artículo\n");
        printf("C. Eliminar Artículo\n");
        printf("D. Aplicar cambios\n");
        printf("F. Gestionar Votación\n");
        printf("G. Listar Artículos\n");
        printf("E. Salir\n");
        printf("Opción: ");
        scanf(" %c", &opcion);
        limpiarBuffer();

        switch (opcion) {
            case 'A':
            case 'a':
                agregarArticulo(congreso, &(ley->articulo));
                break;
            case 'B':
            case 'b':
                printf("Ingrese la sección del artículo a modificar: ");
                scanf("%d", &seccionModificar);
                limpiarBuffer(); // Limpiar el buffer
                if (modificarArticulo(ley->articulo, seccionModificar)) {
                    printf("Artículo modificado correctamente.\n");
                } else {
                    printf("Error: Artículo no encontrado.\n");
                }
                break;
            case 'C':
            case 'c':
                printf("Ingrese la sección del artículo a eliminar: ");
                scanf("%d", &seccionEliminar);
                limpiarBuffer(); // Limpiar el buffer
                if (eliminarArticulo(&(ley->articulo), seccionEliminar) == 1) {
                    printf("Artículo eliminado correctamente.\n");
                } else {
                    printf("Error: Artículo no encontrado.\n");
                }
                break;
            case 'D':
            case 'd':
                copiarCambioATexto(ley->articulo);
                break;
            case 'F':
            case 'f':
                gestionarVotacionArticulo(congreso, ley->articulo);
                break;
            case 'G':
            case 'g':
                mostrarVotacionArticulos(ley);
                break;
            case 'E':
            case 'e':
                printf("Saliendo de la gestión de artículos.\n");
                break;
            default:
                printf("Opción inválida. Intente nuevamente.\n");
        }
    } while (opcion != 'E' && opcion != 'e');
}

void menuComisiones(struct congreso *congreso) {
    char opcion[2];
    char nombre[100];

    while (1) {
        printf("Menu Comisiones.\n"
            "Opcion A: Agregar Comision\n"
            "Opcion B: Borrar Comision\n"
            "Opcion C: Buscar Comision\n"
            "Opcion D: Modificar Comision\n"
            "Opcion E: Listar Comisiones\n"
            "Opcion F: Volver al menu principal\n");

        scanf("%1s", opcion);
        limpiarBuffer();
        opcion[0] = (opcion[0] >= 'a' && opcion[0] <= 'z') ? opcion[0] - ('a' - 'A') : opcion[0];

        switch (opcion[0]) {
            case 'A':
                printf("Funcion: Agregar Comision\n");
                agregarComision(congreso);
                break;
            case 'B':
                printf("Funcion: Borrar Comision\n");
                printf("Ingrese el nombre de la comision a borrar:\n");
                scanf("%99[^\n]", nombre);
                limpiarBuffer(); // Limpiar el buffer
                eliminarComision(congreso, nombre);
                break;
            case 'C':
                printf("Funcion: Buscar Comision\n");
                printf("Ingrese el nombre de la comision buscada:\n");
                scanf("%99[^\n]", nombre);
                limpiarBuffer(); // Limpiar el buffer
                mostrarComisionPorNombre(congreso, nombre);
                break;
            case 'D':
                printf("Funcion: Modificar Comision\n");
                printf("Ingrese el nombre de la comision a modificar:\n");
                scanf("%99[^\n]", nombre);
                limpiarBuffer(); // Limpiar el buffer
                modificarComision(congreso, nombre);
                break;
            case 'E':
                printf("Funcion: Listar Comisiones\n");
                listarComisiones(congreso);
                break;
            case 'F':
                return; // Salir del menú
            default:
                printf("Opcion invalida, por favor intente otra vez.\n");
                break;
        }
    }
}


void mostrarLeyesPorFase(struct nodoProyectoLey *leyes, int faseRequerida) {
    // Verifica si el árbol no está vacío
    if (leyes != NULL) {
        // Verifica si el nodo actual está en la fase requerida
        if (leyes->datos->fase == faseRequerida) {
            printf("Nombre de Ley: %s\n", leyes->datos->nombre);
            printf("Tipo de Ley: %s\n", leyes->datos->tipo);
            printf("Urgencia de Ley: %d\n", leyes->datos->urgencia);
            printf("ID de Ley: %d\n", leyes->datos->idProyecto);
            printf("---------------------------------\n"); // Separador para claridad
        }

        // Llama recursivamente para los hijos izquierdo y derecho
        mostrarLeyesPorFase(leyes->izq, faseRequerida);
        mostrarLeyesPorFase(leyes->der, faseRequerida);
    }
}


int main(void) {
    int flag = 1; // Variable de control del bucle principal
    char opcion;

    // Inicialización del congreso
    struct congreso *congreso = inicializarCongreso();

    // Bucle principal del menú
    while (flag == 1) {
        printf("Opciones:\n"
               "A: Proyectos de Ley.\n"
               "B: Congresistas.\n"
               "C: Comisiones.\n"
               "D: Salir.\n\n");

        opcion = leerOpcion(); // Lee solo un carácter y limpia el buffer

        // Manejo de las opciones seleccionadas
        switch (opcion) {
            case 'A':
                funcionSwitch(opcion, congreso, menuProyectosLey);
            break;
            case 'B':
                funcionSwitch(opcion, congreso, menuCongresistas);
            break;
            case 'C':
                funcionSwitch(opcion, congreso, menuComisiones);
            break;
            case 'D':
                flag = 0; // Cambiar la bandera para salir del bucle
            break;
            default:
                printf("Opción inválida, por favor intente otra vez.\n");
            break;
        }
    }

    // Liberar recursos antes de salir
    liberarCongreso(congreso);
    return 0;
}
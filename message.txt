#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define MAX_CONGRESISTAS 200
#define MAX_SENADORES 50
#define MAX_DIPUTADOS 100
#define MAX_COMISIONES 10

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

void menuProyectosLey();

void menuCongresistas();

void menuComisiones();

void menuArticulos();

struct comision *buscarComision(struct congreso *congreso,char *nombre);

struct proyectoLey *buscarProyectoLeyPorID(struct nodoProyectoLey *raiz, int id);

void funcionSwitch(char opcion, struct congreso *congreso, void (*submenu)(struct congreso *));

struct congresista* comprobarCongresistaEnCongreso(struct congreso *congreso, char *rutBuscado);

/*!!!!!NOTA: LAS FUNCIONES DE VOTACIÓN SE ENCUENTRAN EN EL APARTADO DE LAS FUNCIONES DE PROYECTO DE LEY.!!!!!!!!!!*/

/*TODO: FUNCIONES AUXILIARES---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/
/*TODO---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------*/

void convertirMinusculas(char *cadena) {
    for (int i = 0; cadena[i]; i++) {
        if (cadena[i] >= 'A' && cadena[i] <= 'Z') {
            cadena[i] += 'a' - 'A';
        }
    }
}

//Función para leer un número entero dentro de un rango
int leerEnteroConLimite(const char *mensaje, int min, int max) {
    int valor;
    char input[10];
    while (1) {
        printf("%s (%d-%d): ", mensaje, min, max);
        scanf("%9s", input);
        valor = atoi(input);
        if (valor >= min && valor <= max) break;
        printf("Error: Valor inválido. Debe estar entre %d y %d.\n", min, max);
    }
    return valor;
}

// Función auxiliar para buscar un congresista por RUT en el congreso
struct congresista *buscarCongresistaPorRUT(struct congreso *congreso, const char *rut) {
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
    while ((c = getchar()) != '\n' && c != EOF);
}

char leerOpcion() {
    char opcion;
    scanf("%c", &opcion);
    limpiarBuffer();  // Limpia el buffer para evitar múltiples entradas
    return (opcion >= 'a' && opcion <= 'z') ? opcion - ('a' - 'A') : opcion;  // Convierte a mayúscula si es necesario
}

//TODO: FUNCIÓN DE INICIALIZACIÓN DEL CONGRESO----------------------------------------------------------------------------------------------------------------------------------------------------//
//TODO: ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------//
/*Esta función almacena memoria para todos los datos que debe almacenar "congreso"*/
struct congreso *inicializarCongreso() {
    struct congreso *nuevoCongreso = (struct congreso *) malloc(sizeof(struct congreso));

    if (nuevoCongreso == NULL) {
        return NULL;
    }

    // Inicializa los arreglos para los senadores y diputados
    nuevoCongreso->senadores = (struct congresista **) calloc(MAX_SENADORES, sizeof(struct congresista *));
    nuevoCongreso->diputados = (struct congresista **) calloc(MAX_DIPUTADOS, sizeof(struct congresista *));
    nuevoCongreso->congresistasMixtos = NULL;
    nuevoCongreso->comisionesMixtas = NULL;

    if (nuevoCongreso->senadores == NULL || nuevoCongreso->diputados == NULL) {
        free(nuevoCongreso);
        return NULL;
    }

    // Inicializa los arreglos para las comisiones
    nuevoCongreso->comisiones = (struct comision **) calloc(MAX_COMISIONES, sizeof(struct comision *));

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
        nodo = (struct nodoProyectoLey *) malloc(sizeof(struct nodoProyectoLey));

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

/*Tanto la fase debería ser asignada automáticamente para que siempre sea la primera, seguramente, después de todo...
 * no tiene demasiado sentido que comience desde una fase adelantada
 */


struct proyectoLey *crearProyectoLey(struct congreso *congreso) {
    // Asignar memoria para el proyecto de ley
    struct proyectoLey *nuevoProyecto = (struct proyectoLey *) malloc(sizeof(struct proyectoLey));
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
    nuevoProyecto->nombre = (char *) malloc(strlen(nombre) + 1);
    nuevoProyecto->tipo = (char *) malloc(strlen(tipo) + 1);
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

    struct nodoProyectoLey *nuevoNodo = crearNodoProyectoLey(datos); // crear nodo para el proyecto de ley
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
        return buscarProyectoLeyPorID(raiz->izq, id); // Buscamos en el subarbol izquierdo si es el ID es menor
    } else {
        return buscarProyectoLeyPorID(raiz->der, id); // Buscamos en el subarbol derecho si es el ID es mayor
    }
}

// Encuentra el nodo mínimo en el árbol (usado para encontrar el sucesor en caso de eliminación)
struct nodoProyectoLey *minValorNodo(struct nodoProyectoLey *nodo) {
    struct nodoProyectoLey *actual = nodo;
    while (actual && actual->izq != NULL) {
        actual = actual->izq;
    }
    return actual;
}

// Función para borrar un nodo en el árbol binario de búsqueda
struct nodoProyectoLey *borrarNodoProyectoLey(struct congreso *congreso, struct nodoProyectoLey *raiz, int id) {
    // Base case
    if (raiz == NULL) return raiz;

    // Si el id a eliminar es más pequeño que el id de la raíz, entonces está en el subárbol izquierdo
    if (id < raiz->datos->idProyecto) {
        raiz->izq = borrarNodoProyectoLey(congreso, raiz->izq, id);
    }
    // Si el id a eliminar es mayor que el id de la raíz, entonces está en el subárbol derecho
    else if (id > raiz->datos->idProyecto) {
        raiz->der = borrarNodoProyectoLey(congreso, raiz->der, id);
    }
    // Si el id es el mismo que el id de la raíz, entonces este es el nodo a eliminar
    else {
        // Nodo con solo un hijo o sin hijos
        if (raiz->izq == NULL) {
            struct nodoProyectoLey *temp = raiz->der;
            free(raiz->datos->nombre);
            free(raiz->datos->tipo);
            free(raiz->datos);
            free(raiz);
            return temp;
        } else if (raiz->der == NULL) {
            struct nodoProyectoLey *temp = raiz->izq;
            free(raiz->datos->nombre);
            free(raiz->datos->tipo);
            free(raiz->datos);
            free(raiz);
            return temp;
        }

        // Nodo con dos hijos: obtener el sucesor en el orden (el más pequeño en el subárbol derecho)
        struct nodoProyectoLey *temp = minValorNodo(raiz->der);

        // Copiar el contenido del sucesor al nodo actual
        raiz->datos->idProyecto = temp->datos->idProyecto;
        strcpy(raiz->datos->nombre, temp->datos->nombre);
        strcpy(raiz->datos->tipo, temp->datos->tipo);
        raiz->datos->urgencia = temp->datos->urgencia;
        raiz->datos->fase = temp->datos->fase;

        // Borrar el sucesor
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
                printf("Descripcion de la fase: Iniciativa Legislativa\n");
                break;
            case 2:
                printf("Descripcion de la fase: Camara de Origen\n");
                break;
            case 3:
                printf("Descripcion de la fase: Camara Revisora\n");
                break;
            case 4:
                printf("Descripcion de la fase: Comision Mixta\n");
                break;
            case 5:
                printf("Descripcion de la fase: Promulgacion\n");
                break;
            case 6:
                printf("Descripcion de la fase:Veto presidencial\n");
            case 7:
                printf("Descripcion de la fase: Publicacion y vigencia\n");
                break;
            case 8:
                printf("Descripcion de la fase: Control constitucional\n");
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
    int c;
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
                switch (opcionLista) {
                    case 1: // Lista de votos a favor
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
                        break;

                    case 2: // Lista de votos en contra
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
                        break;
                    default:
                        printf("Opción inválida. Por favor, selecciona una opción válida.\n");
                        break;
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

    recorrerYImprimirProyectos(congreso->raiz);
}

void agregarProyecto(struct proyectoLey *proyecto, struct proyectoLey ***proyectosArray, int *proyectosCount, int *proyectosCapacidad) {
    struct proyectoLey **tempArray;

    if (*proyectosCount >= *proyectosCapacidad) {
        *proyectosCapacidad *= 2;
        tempArray = (struct proyectoLey **) realloc(*proyectosArray, (*proyectosCapacidad) * sizeof(struct proyectoLey *));
        if (tempArray == NULL) {
            printf("Error al redimensionar la memoria.\n");
            exit(1); // Terminar el programa en caso de fallo en realloc
        }
        *proyectosArray = tempArray;
    }
    (*proyectosArray)[(*proyectosCount)++] = proyecto;
}

// Recorrido en orden del árbol binario para agregar los proyectos a la lista
void recorrerArbolEnOrden(struct nodoProyectoLey *nodo, struct proyectoLey ***proyectosArray, int *proyectosCount, int *proyectosCapacidad) {
    if (nodo == NULL) {
        return;
    }
    recorrerArbolEnOrden(nodo->izq, proyectosArray, proyectosCount, proyectosCapacidad);
    agregarProyecto(nodo->datos, proyectosArray, proyectosCount, proyectosCapacidad);
    recorrerArbolEnOrden(nodo->der, proyectosArray, proyectosCount, proyectosCapacidad);
}

// Función de comparación para ordenar los proyectos por urgencia (mayor a menor)
int compararPorUrgencia(const void *a, const void *b) {
    struct proyectoLey *proyectoA = *(struct proyectoLey **)a;
    struct proyectoLey *proyectoB = *(struct proyectoLey **)b;
    return proyectoB->urgencia - proyectoA->urgencia;
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
        printf("Nombre: %s\n", proyecto->nombre);
        printf("Tipo: %s\n", proyecto->tipo);
        printf("ID Proyecto: %d\n", proyecto->idProyecto);
        printf("Urgencia: %d\n", proyecto->urgencia);
        printf("\n");
    }

    // Liberar la memoria del array dinámico
    free(proyectosArray);
}

void modificarProyectoLey(struct congreso *congreso, int idProyecto) {
    struct proyectoLey *proyecto;
    char nombreComision[100]; //se agrega este para el caso del puntero a la comision
    struct comision *comisionEncontrada=NULL; //tambien para el puntero de comision
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
        printf("g. Gestionar Articulos\n");
        printf("h. Asignar Comision\n");
        printf("i. Salir\n");
        printf("Opción: ");

        // Leer opción y limpiar buffer de entrada
        scanf(" %c", &opcion);
        limpiarBuffer();

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
                if (nuevoNombre[strlen(nuevoNombre) - 1] != '\n') {
                    limpiarBuffer();
                } else {
                    nuevoNombre[strcspn(nuevoNombre, "\n")] = '\0'; // Remover salto de línea
                }
                free(proyecto->nombre);
                proyecto->nombre = (char *) malloc(strlen(nuevoNombre) + 1);
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
                if (nuevoTipo[strlen(nuevoTipo) - 1] != '\n') {
                    limpiarBuffer();
                } else {
                    nuevoTipo[strcspn(nuevoTipo, "\n")] = '\0'; // Remover salto de línea
                }
                free(proyecto->tipo);
                proyecto->tipo = (char *) malloc(strlen(nuevoTipo) + 1);
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
                printf("Ingrese el nombre de la comision a asignar: ");
                if (fgets(nombreComision, sizeof(nombreComision), stdin) == NULL) {
                    printf("Error al leer el nombre de la comision.\n");
                    break;
                }
                if (nombreComision[strlen(nombreComision) - 1] != '\n') {
                    limpiarBuffer();
                } else {
                    nombreComision[strcspn(nombreComision, "\n")] = '\0'; // Remover salto de línea
                }
                comisionEncontrada = buscarComision(congreso,nombreComision); // IMPORTANTE INICIALIZAR FUNCION BUSCARCOMISION
                if(comisionEncontrada) {
                    proyecto->comision = comisionEncontrada;
                    printf("Comision asignada correctamente\n");
                }
                else {
                    printf("Error: Comision no asignada\n");
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

    //pregunto primero que los datos recibidos no sean null
    if (datos != NULL) {
        nodo = (struct nodoCongresista *) malloc(sizeof(struct nodoCongresista));

        if (nodo == NULL) {
            //si esto ocurre, hay un error al asignar la memoria
            return NULL;
        }


        nodo->datos = datos; //aqui copio los datos que recibí
        //y le asigno un siguiente para luego insertarlo
        nodo->sig = NULL;
    }
    return nodo;
}

/*

comprobar que exista congresista, lo haré de manera que retorne 0 si NO existe el RUT, o que retorne 1 si existe
la idea es que el RUT sea el buscado, por lo tanto las otras funciones que la llamen deben ingresar el RUT
aunque esto puede estar sujeto a cambios si se desea, quizas recibir el nodo entero para comodidad
recordar que es circular con fantasma, por lo tanto tengo que iniciar el head->sig para el rec y usar do while

*/

int comprobarCongresistaEnComision(struct nodoCongresista *head, char *rutBuscado) {
    struct nodoCongresista *rec = NULL;


    rec = head->sig; //sig porque es fantasma el primero
    do {
        //en este if pregunto que sea distinto de null solo por el nodo fantasma
        if (rec->datos != NULL && strcmp(rec->datos->rut,rutBuscado) == 0) {
            return 1; //se encontró en la lista
        }
        rec = rec->sig;
    }while(rec!=head);

    return 0; //no se encontró en la lista
}

//funcion para recorrer los arreglos, el de diputados o el de senadores correspondientemente
struct congresista* comprobarCongresistaEnCongreso(struct congreso *congreso, char *rutBuscado) {
    int i;
    struct nodoCongresista *head=congreso->congresistasMixtos;
    struct nodoCongresista *rec = NULL;
    struct congresista *congresista = NULL;
    if(rutBuscado!=NULL) {
        for ( i = 0; i<MAX_DIPUTADOS ; i++ ) {
            if (congreso->diputados[i] != NULL)
                if (strcmp(congreso->diputados[i]->rut,rutBuscado) == 0) {
                    congresista = congreso->diputados[i];
                    return congresista;   //el rut se ha encontrado, por lo tanto no se sigue el proceso
                }
            }
        for( i = 0 ; i<MAX_SENADORES ; i++ ) {
            if (congreso->senadores[i] != NULL) {
                if(strcmp(congreso->senadores[i]->rut,rutBuscado) == 0) {
                    congresista = congreso->senadores[i];
                    return congresista;
                }
            }
        }

        if(head != NULL) {
            rec=head;
            while (rec!=NULL) {
                if (rec->datos != NULL && strcmp(rec->datos->rut,rutBuscado) == 0) {
                    congresista = rec->datos;
                    return congresista;
                }
                rec = rec->sig;
            }
        }
    }
    return congresista;           //el rut no se encuentra, se sigue el proceso
}
/*
El crearCongresista va a ser un poco distinto, esta funcion tiene que recibir(scanf) el rut y la ocupacion
al saber la ocupacion eligirá que arreglo recorrer si el de senadores o el de diputados
ahí en ese momento buscará comparando el rut en cada pos del arreglo, si lo encuentra retornará 1
si no es así retornará 0 y dará paso a la copia de datos para ingresarlos en el arreglo

aunque no estoy seguro si la ocupacion fuese el diputado/senador o si es la especializacion

*/

struct congresista *crearCongresista(struct congreso *congreso) {
    struct congresista *nuevoCongresista;                           //esto para guardar los datos del congresista
    char nombre[100],rut[20],ocupacion[20],especializacion[100];    //los datos que se reciben

    nuevoCongresista = (struct congresista *) malloc(sizeof(struct congresista));

    if(nuevoCongresista == NULL) {
        return NULL;                // error al asignar memoria
    }

    //aqui tomará la decision de que arreglo recorrer
    printf("Ingresa el RUT del congresista(12.345.678-9):");
    scanf("%19[^\n]",rut);
    limpiarBuffer();
    if (strlen(rut)>20) {
        printf("El rut ingresado es muy largo, intentelo de nuevo\n");
        limpiarBuffer();
        return NULL;
    }
    printf("Ingrese la ocupacion (senador/diputado):");
    scanf("%19s",ocupacion);
    limpiarBuffer();

    if (comprobarCongresistaEnCongreso(congreso,rut) != NULL) {

        printf("Este congresista ya se encuentra en el sistema\n");
        return NULL; // el congresista ya existe, sin importar si es diputado o senador, anulando la creacion del congresista
    }


    //se escanean los ultimos datos
    printf("Ingrese su especializacion:");
    scanf("%[^\n]",especializacion);
    limpiarBuffer();

    printf("Ingrese su Nombre:");
    scanf("%[^\n]",nombre);
    limpiarBuffer();

    if (strlen(rut) > 20 || strlen(especializacion) > 100 || strlen(nombre) > 100) {

        printf("Uno de los valores ingresados es muy largo, intentelo otra vez\n");
        return NULL;
    }

    //asignacion de memoria
    nuevoCongresista->nombre = (char *)malloc(sizeof(char) * strlen(nombre) + 1);
    nuevoCongresista->rut = (char *)malloc(sizeof(char) * strlen(rut) + 1);
    nuevoCongresista->ocupacion = (char *)malloc(sizeof(char) * strlen(ocupacion) + 1);
    nuevoCongresista->especializacion = (char *)malloc(sizeof(char) * strlen(especializacion) + 1);

    if (nuevoCongresista->nombre == NULL || nuevoCongresista->rut == NULL || nuevoCongresista->ocupacion == NULL || nuevoCongresista->especializacion == NULL) {
        return NULL; //alguno de los valores tuvo un error de asignacion de memoria
    }

    //parte de copiar datos
    strcpy(nuevoCongresista->nombre,nombre);
    strcpy(nuevoCongresista->rut,rut);
    strcpy(nuevoCongresista->ocupacion,ocupacion);
    strcpy(nuevoCongresista->especializacion,especializacion);

    printf("Se ha agregado con exito\n");
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
    int i=0;

    struct congresista *nuevoCongresista = crearCongresista(congreso); //se crea el congresista para insertarlo
    struct nodoCongresista *nuevoNodoCongresista = NULL; //caso de externo
    struct congresista **arreglo = NULL; //decidirá a que arreglo pertenece

    //pregunto que el congresista sea valido
    if (nuevoCongresista != NULL) {
        if (strcmp(nuevoCongresista->ocupacion,"senador") == 0) {
            arreglo = congreso->senadores; //el congresista es un senador
        }
        else if (strcmp(nuevoCongresista->ocupacion,"diputado") == 0) {
            arreglo = congreso->diputados; //el congresista es diputado
        }
        else {
            nuevoNodoCongresista = crearNodoCongresista(congreso->congresistasMixtos,nuevoCongresista);
            if (nuevoNodoCongresista == NULL) {
                printf("Error al asignar memoria\n");
                return;//error al asignar memoria
            }
            nuevoNodoCongresista->datos = nuevoCongresista;
            nuevoNodoCongresista->sig = congreso->congresistasMixtos;
            congreso->congresistasMixtos = nuevoNodoCongresista;

            return; // se agrega
        }

        //se recorre el arreglo elegido validando que se haya elegido uno
        if (arreglo != NULL) {
            if (arreglo == congreso->senadores) {
                while (i < MAX_SENADORES && arreglo[i]!= NULL) {
                    i++; //se busca la 1era posicion vacia
                }
                if (i<MAX_SENADORES) {
                    arreglo[i] = nuevoCongresista; //añade el nuevo congresista
                    return; //se logra asignar sin problema
                }
                else {
                    printf("El arreglo esta lleno, no se ha agregado\n");
                    return;
                }

            }

            if(arreglo == congreso->diputados) {
                while (i < MAX_DIPUTADOS && arreglo[i]!= NULL) {
                    i++; //se busca la 1era posicion vacia
                }
                if (i<MAX_DIPUTADOS) {
                    arreglo[i] = nuevoCongresista; //añade el nuevo congresista
                    return; //se logra asignar sin problema
                }
                else {
                    printf("El arreglo esta lleno, no se ha agregado\n");
                    return;
                }
            }


        }
    }
     //el nuevo congresista era null
}

/*
esta funcion agrega un ya EXISTENTE congresista a la lista de comisiones
voy a darle la comision, pues para acceder a la lista simplemente hago comision->headintegrantes
esto puede cambiar por int para los print
*/

//LA NUEVA PROPUESTA PARA LA FUNCIÓN
void agregarCongresistaEnComision(struct comision *comision, char *rut,struct congresista *congresista) {
    struct nodoCongresista *nuevoNodo = NULL;
    struct nodoCongresista *rec = NULL;

    //PREGUNTO QUE NI UNO DE LOS DOS SEA NULL
    if (comision != NULL) {
        //VERIFICO SI EL NODO FANTASMA YA EXISTE, Y SI NO, CREO UNO


        //VERIFICO QUE EL CONGRESISTA NO ESTÉ EN LA COMISIÓN
        if (comprobarCongresistaEnComision(comision->headIntegrantes, rut) == 0) {
            //CREO Y ENLAZO EL NUEVO NODO
            nuevoNodo = crearNodoCongresista(comision->headIntegrantes, congresista);
            if (nuevoNodo == NULL) {
                return;
            }

            rec = comision->headIntegrantes;
            while (rec->sig != comision->headIntegrantes) {
                rec = rec->sig;
            }
            rec->sig = nuevoNodo;
            nuevoNodo->sig = comision->headIntegrantes;
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
    struct nodoCongresista *rec;  //
    struct nodoCongresista *anterior = NULL; //trabajo con un anterior al ser lista circular

    //puede que la lista esté vacia(no se ha hecho nada) o que esté solo el nodo fantasma
    if (comision->headIntegrantes == NULL || comision->headIntegrantes->sig == comision->headIntegrantes) {
        return; //no existe ni un congresista en la comision, por lo tanto se termina el proceso
    }
    anterior = comision->headIntegrantes; //nodo fantasma, por lo tanto el primero
    rec = comision->headIntegrantes->sig; //nodo despues del fantasma, por lo tanto el segundo

    do {
        if (strcmp(rec->datos->rut,rutQuitado) == 0) {
            anterior->sig = rec->sig;
            return; //se encuentra y se desvincula de la lista
        }
        anterior = rec;
        rec = rec->sig;

    }while(rec != comision->headIntegrantes);

    //no se encontró en la lista

}



void eliminarCongresistaDeCongreso(struct congreso *congreso,char *rutQuitado) {
    struct comision **arreglo = NULL;
    int i,flag=0;
    struct congresista *congresista;
    struct nodoCongresista *rec;
    struct nodoCongresista *recExterno,*ant;

    //pregunto que existan los diputados, porque puede que no hayan comisiones pero quiera eliminar a un congresista
    if (rutQuitado != NULL) {

        congresista = comprobarCongresistaEnCongreso(congreso,rutQuitado);

        if (congresista != NULL) {
            for (i = 0;i<MAX_SENADORES;i++) {
                if (congreso->senadores[i] != NULL && strcmp(congreso->senadores[i]->rut,rutQuitado) == 0) {
                    congreso->senadores[i] = NULL;
                    flag =1;
                }
            }
            if (flag == 0) {
                for (i=0;i<MAX_DIPUTADOS;i++) {
                    if (congreso->diputados[i] != NULL && strcmp(congreso->diputados[i]->rut, rutQuitado) == 0) {
                        congreso->diputados[i] = NULL;
                        flag =1;
                    }
                }
            }
            if(flag == 0 && congreso->congresistasMixtos!=NULL) {
                recExterno =congreso->congresistasMixtos;
                ant = NULL;
                while(recExterno!=NULL && strcmp(recExterno->datos->rut,rutQuitado) != 0) {
                    ant = recExterno;
                    recExterno = recExterno->sig;
                }

                if (recExterno != NULL) {
                    if (ant == NULL) {
                        congreso->congresistasMixtos = recExterno->sig;
                        flag =1;
                    }
                    else {
                        ant->sig = recExterno->sig;
                    }
                }
            }
        }
        //ahora comienzan a borrarse de las comisiones

        //pregunto que existan comisiones
        if (congreso->comisiones != NULL) {

            //asigno el arreglo para comodidad simplemente
            arreglo = congreso->comisiones;

            for(i=0;i<MAX_COMISIONES;i++) {

                if (arreglo[i]!= NULL) {
                    rec = arreglo[i]->headIntegrantes; //recorro el arreglo

                    do {
                        //pregunto si está el congresista en esta comision con la funcion comprobar
                        if (comprobarCongresistaEnComision(rec,rutQuitado) == 1) {

                            //llamo a la funcion eliminar de comision
                            eliminarCongresistaDeComision(arreglo[i],rutQuitado);

                        }
                        rec = rec->sig;
                    }while (rec != arreglo[i]->headIntegrantes);
                }
            }
        }

    }
    if(flag == 1) {
        printf("El congresista fue eliminado\n");
    }
    else {
        printf("El congresista no fue encontrado\n");
    }

     //no se pudo eliminar
}

/*
Funcion para modificar a un congresista
primero tengo que elegir el arreglo en el que está o simplemente recorrer ambos arreglos
en este caso no incluí la modificacion de la ocupación
ya que al cambiar la ocupacion es mejor eliminarlo del arreglo y agregarlo al otro simplemente
*/

void modificarCongresista(struct congreso *congreso,char *rutBuscado) {

    struct congresista *congresista=comprobarCongresistaEnCongreso(congreso,rutBuscado);

    char nombre[100],especializacion[100],rut[20];
    if (congresista != NULL) {
        //escaneo los datos nuevos, solo se copiaran si pasan la siguiente parte, en esta parte irían prints
        printf("Ingrese nuevo Nombre:");
        scanf(" %99[^\n]",nombre); //ingrese nuevo nombre
        printf("Ingrese nueva especializacion:");
        scanf(" %99[^\n]",especializacion); //ingrese nueva especializacion
        printf("Ingrese nuevo rut:");
        scanf(" %19[^\n]",rut); //ingrese nuevo rut
        congresista->rut = (char *)malloc(sizeof(char) * (strlen(rut) + 1));
        congresista->nombre = (char *)malloc(sizeof(char) * (strlen(nombre) + 1));
        congresista->especializacion = (char *)malloc(sizeof(char) * (strlen(especializacion) + 1));

        strcpy(congresista->rut, rut);
        strcpy(congresista->nombre, nombre);
        strcpy(congresista->especializacion, especializacion);
        printf("Fue modificado correctamente\n");
        return;
    }
    printf("Hubo un problema al modificar al congresista\n");
    return; //hubo problemas en la creacion de datos
}

void mostrarCongresista(struct congreso *congreso, char *rutBuscado) {
    struct congresista *congresistaBuscado = NULL;
    //primero lo busco:

    congresistaBuscado = comprobarCongresistaEnCongreso(congreso,rutBuscado);
    if (congresistaBuscado == NULL) {
        printf("No existe el congresista en el sistema\n");
        return;

    }

    if (congresistaBuscado != NULL) {


        printf("Nombre:%s\n",congresistaBuscado->nombre);
        printf("RUT:%s\n",congresistaBuscado->rut);
        printf("Ocupacion:%s\n",congresistaBuscado->ocupacion);
        printf("Especializacion:%s\n\n",congresistaBuscado->especializacion);
    }

}

/*
funcion listar congresista:
esta funcion es un poco redundante, pues llama al mostrarCongresista y esta lo busca de nuevo pero cumple con el requisito de listar
*/
void listarCongresistas(struct congreso *congreso) {
    struct congresista **arregloDiputados = congreso->diputados;
    struct congresista **arregloSenadores = congreso->senadores;
    struct nodoCongresista *rec;
    int i,flagDiputados=0,flagSenadores = 0,flagExternos=0;
    //flag se encarga de mostrar si existen o no en el sistema aunque sea 1 de alguno

    if (congreso == NULL) {
        return;
    }


    printf("Diputados en sistema:\n");
    if(arregloDiputados != NULL) {
        for (i=0;i<MAX_DIPUTADOS;i++) {
            if (arregloDiputados[i]!=NULL) {
                printf("Nombre:%s\n",arregloDiputados[i]->nombre);
                printf("RUT:%s\n\n",arregloDiputados[i]->rut);
                flagDiputados = 1;
            }
        }
    }
    printf("\n\n");
    if(arregloSenadores != NULL) {
        printf("Senadores en sistema:\n");
        for (i=0;i<MAX_SENADORES;i++) {
            if (arregloSenadores[i]!=NULL) {
                printf("Nombre:%s\n",arregloSenadores[i]->nombre);
                printf("RUT:%s\n\n",arregloSenadores[i]->rut);
                flagSenadores = 1;
            }
        }
    }
    printf("\n\n");
    if (congreso->congresistasMixtos != NULL) {
        printf("Externos en sistema:\n");
        rec = congreso->congresistasMixtos;
        while (rec!=NULL) {
            if(rec->datos != NULL) {
                printf("Nombre:%s\n",rec->datos->nombre);
                printf("RUT:%s\n\n",rec->datos->rut);
                flagExternos = 1;
            }
            rec = rec->sig;
        }
    }

    if (flagDiputados == 0 && flagSenadores == 0 && flagExternos == 0) {
        printf("El sistema no tiene congresistas \n");
    }
}


//FUNCIONES PARA CREAR LOS CONGRESISTAS QUE NO SON DIPUTADOS NI SENADORES


//crear nodo para la lista doble

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
    nuevoNodo->datos = datos;           // Asignar la comisión al nodo
    nuevoNodo->sig = NULL;             // Inicializar el siguiente nodo como NULL

    return nuevoNodo; // Retornar el nuevo nodo
}

struct comision *buscarComision(struct congreso *congreso,char *nombre) {
    int i;
    struct nodoComision *rec = NULL;
    struct comision *buscado = NULL;

    for(i=0;i<MAX_COMISIONES;i++) {
        if (congreso->comisiones[i] != NULL && strcmp(congreso->comisiones[i]->nombre,nombre)==0) {
            buscado = congreso->comisiones[i];
            return buscado;
        }
    }
    rec = congreso->comisionesMixtas;
    while (rec!=NULL) {
        if (strcmp(rec->datos->nombre,nombre)==0) {
            buscado = rec->datos;
            return buscado;
        }
        rec = rec->sig;
    }
    return buscado;
}

struct comision *crearComision(struct congreso *congreso) {
    struct comision *nuevaComision = NULL;
    struct nodoCongresista *fantasma = NULL;
    char nombre[100],tipo[50],descripicion[256];
    nuevaComision = (struct comision*)malloc(sizeof(struct comision));
    if (nuevaComision == NULL) {
        printf("Error al asignar memoria\n");
        return NULL;
    }

    printf("Ingresa el nombre de la comision:\n");
    scanf("%99[^\n]", nombre);
    limpiarBuffer();

    if (buscarComision(congreso,nombre) != NULL) {
        printf("La comision ya existe en el sistema\n");
        free(nuevaComision);
        return NULL;
    }

    printf("Ingresa descripcion de la comision:\n");
    scanf("%255[^\n]", descripicion);
    limpiarBuffer();

    printf("Ingresa el tipo de la comision(senadores/diputados/otros):\n");
    scanf("%49[^\n]", tipo);
    limpiarBuffer();

    convertirMinusculas(tipo);

    nuevaComision->nombre = (char*)malloc(sizeof(char)*strlen(nombre) + 1);
    nuevaComision->descripcion = (char*)malloc(sizeof(char)*strlen(descripicion) + 1);
    nuevaComision->tipo = (char*)malloc(sizeof(char)*strlen(tipo) + 1);

    if (nuevaComision->nombre == NULL || nuevaComision->descripcion == NULL || nuevaComision->tipo == NULL) {
        printf("error al asignar memoria de uno de los campos\n");
        return NULL;
    }

    strcpy(nuevaComision->nombre,nombre);
    strcpy(nuevaComision->descripcion,descripicion);
    strcpy(nuevaComision->tipo,tipo);



    fantasma = (struct nodoCongresista *)malloc(sizeof(struct nodoCongresista));
    if (fantasma == NULL) {
        printf("error al crear el fantasma");
        return NULL;
    }
    fantasma->datos = NULL;
    fantasma->sig = fantasma;
    nuevaComision->headIntegrantes = fantasma;

    return nuevaComision;

}


void agregarComision(struct congreso *congreso) {
    int i;
    struct comision *nuevaComision = crearComision(congreso);
    struct nodoComision *nuevoNodo = NULL;

    if (nuevaComision == NULL) {
        printf("Error al crear la nueva comision\n");
        return;
    }
    if (strcmp(nuevaComision->tipo,"senadores")==0 || strcmp(nuevaComision->tipo,"diputados") == 0) {
        for (i=0;i<MAX_COMISIONES;i++) {
            if (congreso->comisiones[i]==NULL) {
                congreso->comisiones[i] = nuevaComision;
                printf("Comision agregada al arreglo de comisiones\n");
                return;
            }
        }
        if (i == MAX_COMISIONES || i > MAX_COMISIONES) {
            printf("Error al asignar en el arreglo");
            return;
        }
    }
    else {
        nuevoNodo = crearNodoComision(nuevaComision);
        if (nuevoNodo == NULL) {
            printf("Error al asignar memoria al nodo\n");
            return;
        }

        nuevoNodo->sig = congreso->comisionesMixtas;
        congreso->comisionesMixtas = nuevoNodo;
        printf("Comision agregada como mixta\n");
        return;
    }
}

void eliminarComision(struct congreso *congreso, char *nombre) {
    int i;
    struct comision *buscada = buscarComision(congreso,nombre);
    struct nodoComision *rec,*ant;
    if (buscada == NULL) {
        printf("Error al eliminar el comision\n");
        return;
    }

    for (i=0;i<MAX_COMISIONES;i++) {
        if (congreso->comisiones[i]== buscada) {
            free (congreso->comisiones[i]->nombre);
            free (congreso->comisiones[i]->descripcion);
            free (congreso->comisiones[i]->tipo);
            free (congreso->comisiones[i]->headIntegrantes);
            congreso->comisiones[i] = NULL;
            printf("Comision eliminada correctamente\n");
            return;
        }

    }
    //si no está en arreglo se busca en lista
    rec = congreso->comisionesMixtas;
    ant = NULL;

    while (rec != NULL) {
        if(rec->datos == buscada) {
            //caso primer nodo
            if (ant==NULL) {
                congreso->comisionesMixtas = rec->sig;
            }
            //otro caso
            else {
                ant->sig = rec->sig;
            }
            free(rec->datos->nombre);
            free(rec->datos->descripcion);
            free(rec->datos->tipo);
            free(rec->datos);
            printf("Eliminado Correctamente de lista\n");
            return;
        }
        ant = rec;
        rec = rec->sig;
    }
    printf("Error al eliminar la comision\n");
}

void modificarComision(struct congreso *congreso, char *nombre) {
    struct congresista *nuevoCongresista = NULL;
    struct comision *comisionAModificar = buscarComision(congreso, nombre);
    int opcion,subOpcion; //genero una sub opcion para el agregar y eliminar congresista de comision al igual que el rut
    char nuevoNombre[100],nuevoTipo[50],nuevaDescripcion[256],rut[20];

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
        case 1:
            printf("Ingrese el nuevo nombre de la comision:\n");
            scanf("%99[^\n]", nuevoNombre);
            limpiarBuffer();

            if (buscarComision(congreso, nuevoNombre) != NULL) {
                printf("El nombre ya está en uso. Modificación cancelada.\n");
                return;
            }

            free(comisionAModificar->nombre);  // Liberar la memoria actual del nombre
            comisionAModificar->nombre = (char*)malloc(sizeof(char) * strlen(nuevoNombre) + 1);
            if (comisionAModificar->nombre == NULL) {
                printf("Error al asignar memoria para el nuevo nombre.\n");
                return;
            }
            strcpy(comisionAModificar->nombre, nuevoNombre);
            printf("Nombre de la comisión actualizado correctamente.\n");
            break;

        case 2:
            printf("Ingrese el nuevo tipo de la comisión (senadores/diputados/otros):\n");
            scanf("%49[^\n]", nuevoTipo);
            limpiarBuffer();

            free(comisionAModificar->tipo);  // Liberar la memoria actual del tipo
            comisionAModificar->tipo = (char*)malloc(strlen(nuevoTipo) + 1);
            if (comisionAModificar->tipo == NULL) {
                printf("Error al asignar memoria para el nuevo tipo.\n");
                return;
            }
            strcpy(comisionAModificar->tipo, nuevoTipo);
            printf("Tipo de la comisión actualizado correctamente.\n");
            break;

        case 3:
            printf("Ingrese la nueva descripción de la comisión:\n");
            scanf("%255[^\n]", nuevaDescripcion);
            limpiarBuffer();

            free(comisionAModificar->descripcion);  // Liberar la memoria actual de la descripción
            comisionAModificar->descripcion = (char*)malloc(strlen(nuevaDescripcion) + 1);
            if (comisionAModificar->descripcion == NULL) {
                printf("Error al asignar memoria para la nueva descripción.\n");
                return;
            }
            strcpy(comisionAModificar->descripcion, nuevaDescripcion);
            printf("Descripción de la comisión actualizada correctamente.\n");
            break;

        case 4:
            printf("Menu de modificar congresistas en comision\n");
            printf("Elija una opcion:\n");
            printf("1. Agregar Congresista a comision\n");
            printf("2. Eliminar Congresista a comision\n");
            scanf("%d",&subOpcion);
            limpiarBuffer();
            switch (subOpcion) {
                case 1:
                    printf("Ingrese el RUT del congresista a agregar\n");
                    scanf("%19s",rut);
                    limpiarBuffer();
                    nuevoCongresista = comprobarCongresistaEnCongreso(congreso,rut);
                    if(nuevoCongresista != NULL) {
                        if(comprobarCongresistaEnComision(comisionAModificar->headIntegrantes,rut)==1) {
                            printf("Este congresista ya se encuentra en la lista");

                        }
                        else {
                            agregarCongresistaEnComision(comisionAModificar,rut,nuevoCongresista);
                            printf("Agregado correctamente");
                        }
                    }
                break;

                case 2:
                    printf("Ingrese el RUT del congresista a agregar\n");
                    scanf("%19s",rut);
                    limpiarBuffer();
                    if (comprobarCongresistaEnComision(comisionAModificar->headIntegrantes,rut)==1) {
                        eliminarCongresistaDeComision(comisionAModificar,rut);
                        printf("Congresista eliminado\n");
                    }
                    else {
                        printf("No se encontro al congresista en la comision\n");
                    }
                    break;
                default:
                    printf("Opcion no valida\n");
                break;
            }

            break;
        default:
            printf("Opción no válida.\n");
            break;
    }
}


void mostrarComisionPorNombre(struct congreso *congreso,char *nombre) {
    struct comision *Buscada = buscarComision(congreso,nombre);
    struct nodoCongresista *rec;
    if (Buscada == NULL) {
        printf("No existe una comision con este nombre\n");
        return;
    }

    printf("Comision encontrada:\n\n");
    printf("Nombre = %s\n", Buscada->nombre);
    printf("Descripcion = %s\n", Buscada->descripcion);
    printf("Tipo = %s\n", Buscada->tipo);

    rec = Buscada->headIntegrantes;
    if (rec == NULL) {
        printf("No hay integrantes en la comision\n");
    }
    else {
        printf("Integrantes de comision:\n\n");
        rec = rec->sig; //esto es por fantasma
        do {
            if(rec->datos != NULL) {
                printf("-Nombre:%s\n",rec->datos->nombre);
                printf("-Ocupacion:%s\n\n",rec->datos->ocupacion);
            }
            rec =rec->sig;
        }while (rec!=Buscada->headIntegrantes);
    }

}

void listarComisiones(struct congreso *congreso) {
    int i;
    struct nodoComision *rec;
    printf("Comisiones de senadores y diputados:\n");
    for (i=0;i<MAX_COMISIONES;i++) {
        if (congreso->comisiones[i]!=NULL) {
            printf("Nombre:%s\n", congreso->comisiones[i]->nombre);
            printf("Descripcion:%s\n", congreso->comisiones[i]->descripcion);
            printf("Tipo:%s\n", congreso->comisiones[i]->tipo);
            printf("\n");
        }
    }
    printf("\nComisiones mixtas:\n");
    rec = congreso->comisionesMixtas;
    while (rec!= NULL) {
        printf("Nombre:%s\n",rec->datos->nombre);
        printf("Descripcion:%s\n",rec->datos->descripcion);
        printf("Tipo:%s\n",rec->datos->tipo);
        printf("\n");

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
                // Asignar memoria para "texto" y copiar el contenido de "cambio"
                if (rec->datos->texto != NULL) {
                    free(rec->datos->texto);  // Liberar memoria previa si existe
                }
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

struct nodoArticulo *crearNodoArticulo(struct nodoArticulo *head, struct articulo *datos) {

    struct nodoArticulo *nodo = NULL;

    //pregunto primero que los datos recibidos no sean null
    if (datos != NULL) {


        nodo = (struct nodoArticulo *) malloc(sizeof(struct nodoArticulo));

        if (nodo == NULL) {
            //si esto ocurre, hay un error al asignar la memoria
            return NULL;
        }

        nodo->datos = datos; //aqui copio los datos que recibí

        //y le asigno un siguiente para luego insertarlo
        nodo->sig = NULL;
        nodo->ant = NULL;
    }
    return nodo;
}

/*

comprobar que exista articulo, lo haré de manera que retorne 0 si NO existe el articulo, o que retorne 1 si existe
la idea es que la seccion sea el buscado, por lo tanto las otras funciones que la llamen deben ingresar la seccion
aunque esto puede estar sujeto a cambios si se desea, quizas recibir el nodo entero para comodidad

*/
int comprobarArticulo(struct nodoArticulo *head,int buscado) {
    struct nodoArticulo *rec;

    if (head == NULL) {
        return 0;
    }

    //existe la lista
    if (head != NULL) {
        rec = head;
        while (rec!= NULL) {
            printf("se cae en comprobar while");
            if(rec->datos->seccion == buscado) {
                printf("entra al if");
                return 1; //se encuentra en la lista, por lo tanto no se inserta
            }
            printf("se cae despues del if");
            rec = rec->sig;
        }
    }
    return 0; //no se encuentra en la lista, por lo que se pasa a la insercion
}



/*
Agregar el nuevo articulo, llamará a 2 funciones, la de crear el nodo y la de crear el nuevo articulo
la idea es que primero se creen los datos del articulo y luego se cree el nodo, la funcion crear nodo articulo
recibirá los datos de la función crear articulo

PD: todos los prints serán eliminados posterior a la creacion del main
*/

struct articulo *crearArticulo(struct nodoArticulo *lista) {
    struct articulo *NuevoArticulo;
    char nombre[100],texto[256],cambio[256];
    int seccion;
    NuevoArticulo = (struct articulo *)malloc(sizeof(struct articulo));

    if (NuevoArticulo == NULL) {
        return NULL; //significa que hubo un error en la asignacion de memoria
    }
    printf("Ingresa el numero de seccion del articulo:");
    scanf("%d",&seccion); //aqui recibo primero el numero de seccion para utilizar el comprobar que no se repita

    NuevoArticulo->seccion = seccion;
    NuevoArticulo->voto = NULL;


    printf("Ingrese el nombre:");
    scanf(" %[^\n]",nombre);
    limpiarBuffer();

    printf("Ingrese la descripcion del articulo:");
    scanf(" %[^\n]",texto);
    limpiarBuffer();

    printf("Ingrese los cambios:");

    scanf(" %[^\n]",cambio);
    limpiarBuffer();

    if (strlen(nombre)>100 || strlen(texto)>256 || strlen(cambio)>256) {
        printf("Uno de los valores ingresados es muy grande\n");
        return NULL;
    }

    //no creo que haya mucha diferencia a lo que es un malloc
    NuevoArticulo->nombre = (char *)malloc(sizeof(char)*strlen(nombre)+1);
    NuevoArticulo->texto = (char *)malloc(sizeof(char)*strlen(texto)+1);
    NuevoArticulo->cambio = ((char *)malloc(sizeof(char)*strlen(cambio)+1));

    if (NuevoArticulo->nombre == NULL || NuevoArticulo->texto == NULL || NuevoArticulo->cambio == NULL) {
        return NULL; //en alguno de los procesos hubo un error de asignacion de memoria
    }

    strcpy(NuevoArticulo->nombre,nombre);
    strcpy(NuevoArticulo->texto,texto);
    strcpy(NuevoArticulo->cambio,cambio);


    return NuevoArticulo;

}

void agregarArticulo(struct congreso *congreso,struct nodoArticulo **lista) {
    printf("Depuración: Valor de *lista al entrar en agregarArticulo: %p\n", (void *)*lista);
    struct nodoArticulo *NuevoArticulo;
    struct nodoArticulo *rec;
    struct articulo *datos;

    datos = crearArticulo(*lista);
    if (datos == NULL) {
        printf("Error al asignar datos\n");
        return;
    }
    NuevoArticulo = crearNodoArticulo(*lista,datos);
    //pregunto que no hayan errores al crear el nodo
    if (NuevoArticulo != NULL) {
        //printf("Depuración: Valor de *lista al entrar en agregarArticulo: %p\n", (void *)*lista);
        //a partir de aqui pregunto los tipos de caso, cuando no hay nodos, cuando hay un nodo, etc
        if (*lista == NULL) {
            (*lista) = NuevoArticulo; //la lista estaba vacia, se agrega sin problemas
            return;
        }
        //la lista no está vacia
        else {
            if(comprobarArticulo(*lista,datos->seccion) == 0) {
                rec = *lista;
                printf("se cae antes de llegar a la ultima pos");
                //llego a la ultima posicion de la lista
                while (rec->sig != NULL) {
                    rec = rec->sig;
                }
                rec->sig = NuevoArticulo; //llego al final y lo agrego
                NuevoArticulo->ant = rec;
                printf("Agregado correctamente.\n");
                return; //agregado correctamente
            }
        }

    }
     //no se cumple los requisitos, no se agrega
}

/*
El eliminar articulo tambien será int para que en la consola sea mas facil poner algo como: articulo eliminado, o error
si devuelve 1, ha sido un exito la eliminacion, si retorna 0, no se encontró en la lista
la funcion recibe la lista de articulos, la idea es que se seleccione la ley y a partir de ahi se elimine el articulo
*/


int eliminarArticulo(struct nodoArticulo **lista,int seccionEliminada) {
    struct nodoArticulo *rec;

    //la lista existe
    if(*lista != NULL) {
        rec = *lista;

        //caso 1: el articulo está en la primera posicion
        if (seccionEliminada == (*lista)->datos->seccion) {
            *lista = (*lista)->sig;
            return 1; //se ha encontrado y eliminado el articulo
        }

        //caso 2: se encuentra en cualquier posicion de la lista
        while(rec->sig != NULL) {

            if(seccionEliminada == rec->sig->datos->seccion) {
                rec->sig = rec->sig->sig;
                return 1; //se ha encontrado y eliminado el articulo
            }

            rec =rec->sig;
        }
    }
    return 0; //no se encontró el articulo
}

//return 1: modificado de forma correcta return 0: no se pudo modificar

int modificarArticulo(struct nodoArticulo *articulos, int seccionModificada) {
    struct nodoArticulo *rec;
    struct articulo *articuloBuscado = NULL;
    char nombre[100],texto[256],cambio[256];

    //se escanean los datos nuevos
    scanf("%[^\n]",nombre);
    scanf("%[^\n]",texto);
    scanf("%[^\n]",cambio);

    if (strlen(texto) >256 || strlen(cambio)>256) {
        printf("uno de los valores ingresados es muy largo");
        return 0;
    }
    //que la lista no sea null
    if (articulos != NULL) {
        rec = articulos;

        //recorro y busco el especifico
        while (rec != NULL) {
            //si se encuentra el que se modifica
            if (rec->datos->seccion == seccionModificada) {
                articuloBuscado = rec->datos; //se copia la info del articulo encontrado

                //ahora almaceno memoria
                articuloBuscado->nombre = (char *)malloc(sizeof(char) * strlen(nombre) + 1);
                articuloBuscado->texto = (char *)malloc(sizeof(char) * strlen(texto) + 1);
                articuloBuscado->cambio = (char *)malloc(sizeof(char) * strlen(cambio) + 1);

                //copio los datos
                strcpy(articuloBuscado->nombre,nombre);
                strcpy(articuloBuscado->texto,texto);
                strcpy(articuloBuscado->cambio,cambio);

                return 1; //modificado correctamente
            }
            rec = rec->sig;
        }

    }
    return 0; //no se logra modificar
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
            printf("Seleccionaste la opcion E. Cerrando el menu.");
        default:
            printf("Opcion invalida, por favor intente otra vez.\n");
            break;
    }
}

void menuProyectosLey(struct congreso *congreso) {
    char opcion[2];
    int id;
    while (1) {
        printf("Menu Proyectos de Ley.\n"
            "Opcion A: Agregar nuevo Proyecto de Ley\n"
            "Opcion B: Borrar Proyecto de Ley\n"
            "Opcion C: Mostrar Proyecto de Ley\n"
            "Opcion D: Modificar Proyecto de Ley\n"
            "Opcion E: Listar Proyectos de Ley\n"
            "Opcion F: Mostrar Proyectos en Orden de Urgencia\n"
            "Opcion G: Volver al menu principal\n");

        scanf("%1s", opcion);
        limpiarBuffer();
        opcion[0] = (opcion[0] >= 'a' && opcion[0] <= 'z') ? opcion[0] - ('a' - 'A') : opcion[0];

        switch (opcion[0]) {
            case 'A':
                printf("Funcion: Agregar nuevo Proyecto de Ley\n");
                agregarNodoProyectoLey(congreso);
                break;
            case 'B':
                printf("Funcion: Borrar Proyecto de Ley\n");
                id = leerEnteroConLimite("ingrese la ID del Proyecto que quiere borrar", 10000000, 99999999);
                borrarProyectoLey(congreso, id);
                break;
            case 'C':
                printf("Funcion: Mostrar Proyecto de Ley\n");
                id = leerEnteroConLimite("ingrese la ID del Proyecto que quiere buscar", 10000000, 99999999);
                buscarYMostrarProyectoLey(congreso, id);
                break;
            case 'D':
                printf("Funcion: Modificar Proyecto de Ley\n");
                id = leerEnteroConLimite("ingrese la ID del Proyecto que quiere modificar", 10000000, 99999999);
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
                return;
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
        opcion[0] = (opcion[0] >= 'a' && opcion[0] <= 'z') ? opcion[0] - ('a' - 'A') : opcion[0];

        switch (opcion[0]) {
            case 'A':
                printf("Funcion: Agregar Congresista\n");
                agregarCongresistaEnCongreso(congreso);
            break;
            case 'B':
                printf("Funcion: Borrar Congresista\n Ingrese el rut del congresista a eliminar:");
                scanf("%19[^\n]",rut);
                limpiarBuffer();
                eliminarCongresistaDeCongreso(congreso,rut);
            break;
            case 'C':
                printf("Funcion: Buscar Congresista\n");
                printf("Ingrese el rut del congresista a buscar: ");
                scanf("%19[^\n]",rut);
                limpiarBuffer();
                mostrarCongresista(congreso,rut);
            break;
            case 'D':
                printf("Funcion: Modificar Congresista\n");
                printf("ingrese el rut del congresista a modificar:\n");
                scanf(" %19[^\n]",rut);
                limpiarBuffer();
                modificarCongresista(congreso,rut);
            break;
            case 'E':
                printf("Funcion: Listar Congresistas\n");
                listarCongresistas(congreso);
            break;
            case 'F':
                return;
            default:
                printf("Opcion invalida, por favor intente otra vez.\n");
            break;
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
                rec->datos->voto->favor = NULL;
                rec->datos->voto->contra = NULL;
            }

            // Preguntar al usuario si desea añadir un congresista a favor o en contra
            printf("¿Desea añadir un voto (F)avor o (C)ontra? ");
            scanf(" %c", &opcionVoto);
            limpiarBuffer();

            if (opcionVoto == 'F' || opcionVoto == 'f' || opcionVoto == 'C' || opcionVoto == 'c') {
                printf("Ingrese el RUT del congresista (formato 12.345.678-K): ");
                scanf("%19[^\n]",rut);
                limpiarBuffer();

                // Buscar al congresista en el congreso usando el RUT
                struct congresista *congresista = comprobarCongresistaEnCongreso(congreso, rut);
                if (congresista == NULL) {
                    printf("Error: Congresista con RUT %s no encontrado.\n", rut);
                    return;
                }

                // Crear un nuevo nodo de congresista para añadirlo a la votación
                struct nodoCongresista *nuevoNodo = (struct nodoCongresista *)malloc(sizeof(struct nodoCongresista));
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
                } else if (opcionVoto == 'C' || opcionVoto == 'c') {
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

        articuloActual = articuloActual->sig;
    }
}

void menuArticulos(struct congreso *congreso, struct proyectoLey *ley) {
    char opcion;
    int seccionEliminar;
    int seccionModificar;

    do {
        printf("Seleccione la acción sobre los artículos:\n");
        printf("A. Agregar Articulo\n");
        printf("B. Modificar Articulo\n");
        printf("C. Eliminar Articulo\n");
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
            scanf("%99[^\n]",nombre);
            eliminarComision(congreso,nombre);
            break;
            case 'C':
                printf("Funcion: Buscar Comision\n");
            printf("Ingrese el nombre de la comision buscada:\n");
            scanf("%99[^\n]",nombre);
            mostrarComisionPorNombre(congreso,nombre);
            break;
            case 'D':
                printf("Funcion: Modificar Comision\n");
            printf("Ingrese el nombre de la comision a modificar:\n");
            scanf("%99[^\n]",nombre);
            modificarComision(congreso,nombre);
            break;
            case 'E':
                printf("Funcion: Listar Comisiones\n");
            listarComisiones(congreso);
            break;
            case 'F':
                return;
            default:
                printf("Opcion invalida, por favor intente otra vez.\n");
            break;
        }
    }
}

void mostrarLeyesPorFase(struct nodoProyectoLey *leyes,int faseRequerida) {
    //pregunto si el arbol está creado
    if (leyes != NULL) {

        //ahora pregunto si el actual está en la fase requerida
        if (leyes->datos->fase == faseRequerida) {
            printf("Nombre de Ley: %s \n",leyes->datos->nombre);
            printf("Tipo de Ley: %s \n",leyes->datos->tipo);
            printf("urgencia de Ley: %d \n",leyes->datos->urgencia);
            printf("ID de Ley: %d \n",leyes->datos->idProyecto);
            
        }
        mostrarLeyesPorFase(leyes->izq,faseRequerida);
        mostrarLeyesPorFase(leyes->der,faseRequerida);
    }

}

void menuFuncionesExtras(struct congreso *congreso) {
    int fase; //para la funcion mostrar leyes por fase
    char opcion[2];
    while(1){
        printf("Menu funcion extra mostrar leyes por fase:\n"
                "Opcion A: Mostrar leyes por fase\n"
                "Opcion B: Volver al menu principal\n");

        scanf("%1s",opcion);
        limpiarBuffer();
        opcion[0] = (opcion[0] >= 'a' && opcion[0] <= 'z') ? opcion[0] - ('a' - 'A') : opcion[0];

        switch(opcion[0]){
            case 'A':{
                printf("Fase 1: Iniciativa Legislativa\n"
                        "Fase 2: Camara de Origen\n"
                        "Fase 3:Camara Revisora\n"
                        "Fase 4:Comision Mixta\n"
                        "Fase 5:Promulgacion\n"
                        "Fase 6:Veto Presidencial\n"
                        "Fase 7:Publicacion y vigencia\n"
                        "Fase 8:Control constitucional\n"
                        "Ingrese la fase que desea mostrar:");
                scanf("%d",&fase);
                mostrarLeyesPorFase(congreso->raiz,fase);
                break;
            }
            case 'B':{
                return;
            }

            default:
                printf("Opción inválida, por favor intente otra vez.\n");
            break;
        }
    }
}


int main(void)
{
    int flag = 1;
    char opcion;

    struct congreso *congreso = NULL;
    congreso = inicializarCongreso();

    while (flag == 1) {
        printf("Opcion A: Proyectos de Ley.\n"
            "Opcion B: Congresistas.\n"
            "Opcion C: Comisiones.\n"
            "Opcion D: Funciones extras.\n"
            "Opcion E: Salir\n\n");

        opcion = leerOpcion(); // Lee solo un carácter y limpia el buffer

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
                funcionSwitch(opcion, congreso, menuFuncionesExtras);
            break;
            case 'E':
                flag = 0;
            break;
            default:
                printf("Opcion invalida, por favor intente otra vez.\n");
            break;
        }
    }

    liberarCongreso(congreso);
    return 0;
}

# ExamenFinalOptativa
import pygame
import random
import time

# Inicializar Pygame
pygame.init()

# Configurar la pantalla
ANCHO = 800
ALTO = 600
pantalla = pygame.display.set_mode((ANCHO, ALTO))
pygame.display.set_caption("Juego de Carreras")

# Colores
NEGRO = (0, 0, 0)
BLANCO = (255, 255, 255)
ROJO = (255, 0, 0)
GRIS = (100, 100, 100)
AZUL = (0, 0, 255)

# Cargar y redimensionar imágenes
auto_img = pygame.image.load("auto.png")
auto_img = pygame.transform.scale(auto_img, (100, 150))
obstaculo_img = pygame.image.load("obstaculo.png")
obstaculo_img = pygame.transform.scale(obstaculo_img, (100, 150))
fondo_img = pygame.image.load("fondo.png")
fondo_img = pygame.transform.scale(fondo_img, (ANCHO, ALTO))

# Tamaños de las imágenes
auto_tam = auto_img.get_size()
obstaculo_tam = obstaculo_img.get_size()

# Configuración de juego
velocidad_inicial = 20
velocidad_maxima = 35
velocidad_incremento = 0.8
espacio_seguro_inicial = 100
espacio_seguro_minimo = 50
densidad_inicial = 0.02
densidad_maxima = 0.06
densidad_incremento = 0.001
tiempo_limite = 60  # segundos

# Configurar el auto del jugador
auto_x = ANCHO // 2
auto_y = ALTO - auto_tam[1] - 10
auto_vel = velocidad_inicial

# Configurar obstáculos
obstaculo_vel = velocidad_inicial
obstaculos = []
espacio_seguro = espacio_seguro_inicial
densidad = densidad_inicial

# Reloj para controlar la velocidad de actualización
reloj = pygame.time.Clock()

# Tiempo restante
tiempo_restante = tiempo_limite

# Cargar sonidos
sonido_colision = pygame.mixer.Sound("colision.wav")
sonido_motor = pygame.mixer.Sound("motor.wav")  # Sonido del motor del auto

# Fuente para la puntuación y el tiempo
fuente = pygame.font.SysFont(None, 36)
fuente_grande = pygame.font.SysFont(None, 72)

# Puntuación inicial
puntuacion = 0

# Variables para gestionar turnos y estado del juego
turno_jugador = 1
juego_terminado = False
multijugador_terminado = False
modo_juego = "solo"  # Puede ser "solo" o "multijugador"

# Función para crear un nuevo obstáculo
def crear_obstaculo():
    x = random.randint(0, ANCHO - obstaculo_tam[0])
    y = -obstaculo_tam[1]
    obstaculos.append(pygame.Rect(x, y, obstaculo_tam[0], obstaculo_tam[1]))

# Función para mostrar la puntuación en la pantalla
def mostrar_puntuacion(pantalla, puntuacion):
    texto = fuente.render(f"Puntuación: {puntuacion}", True, BLANCO)
    pantalla.blit(texto, (10, 10))

# Función para mostrar el tiempo restante en la pantalla
def mostrar_tiempo(pantalla, tiempo):
    texto = fuente.render(f"Tiempo: {tiempo:.1f}", True, BLANCO)
    pantalla.blit(texto, (ANCHO - 150, 10))

# Función para mostrar el mensaje de choque
def mostrar_mensaje_choque(pantalla, puntuacion):
    pantalla.fill(NEGRO)
    mensaje = fuente_grande.render("¡Has chocado!", True, ROJO)
    pantalla.blit(mensaje, (ANCHO // 2 - mensaje.get_width() // 2, ALTO // 2 - mensaje.get_height() // 2))
    mensaje_puntuacion = fuente.render(f"Puntuación final: {puntuacion}", True, BLANCO)
    pantalla.blit(mensaje_puntuacion, (ANCHO // 2 - mensaje_puntuacion.get_width() // 2, ALTO // 2 + 50))
    pygame.display.flip()

# Función para mostrar la pantalla de inicio
def pantalla_inicio():
    pantalla.blit(fondo_img, (0, 0))
    titulo = fuente_grande.render("Juego de Carreras", True, BLANCO)
    pantalla.blit(titulo, (ANCHO // 2 - titulo.get_width() // 2, ALTO // 2 - 150))
    
    boton_un_jugador_rect = pygame.Rect(ANCHO // 2 - 100, ALTO // 2 - 50, 200, 50)
    pygame.draw.rect(pantalla, GRIS, boton_un_jugador_rect)
    texto_boton_un_jugador = fuente.render("Un Jugador", True, BLANCO)
    pantalla.blit(texto_boton_un_jugador, (boton_un_jugador_rect.x + 25, boton_un_jugador_rect.y + 10))
    
    boton_multijugador_rect = pygame.Rect(ANCHO // 2 - 100, ALTO // 2 + 10, 200, 50)
    pygame.draw.rect(pantalla, GRIS, boton_multijugador_rect)
    texto_boton_multijugador = fuente.render("Multijugador", True, BLANCO)
    pantalla.blit(texto_boton_multijugador, (boton_multijugador_rect.x + 15, boton_multijugador_rect.y + 10))
    
    boton_salir_rect = pygame.Rect(ANCHO // 2 - 100, ALTO // 2 + 70, 200, 50)
    pygame.draw.rect(pantalla, GRIS, boton_salir_rect)
    texto_boton_salir = fuente.render("Salir", True, BLANCO)
    pantalla.blit(texto_boton_salir, (boton_salir_rect.x + 75, boton_salir_rect.y + 10))
    
    pygame.display.flip()
    return boton_un_jugador_rect, boton_multijugador_rect, boton_salir_rect

# Función para reiniciar el juego
def reiniciar_juego():
    global auto_x, auto_y, obstaculos, puntuacion, tiempo_restante, auto_vel, obstaculo_vel, espacio_seguro, densidad, sonido_motor
    auto_x = ANCHO // 2
    auto_y = ALTO - auto_tam[1] - 10
    obstaculos = []
    puntuacion = 0
    tiempo_restante = tiempo_limite
    auto_vel = velocidad_inicial
    obstaculo_vel = velocidad_inicial
    espacio_seguro = espacio_seguro_inicial
    densidad = densidad_inicial
    sonido_motor.play(-1)

# Función para mostrar un mensaje en la pantalla durante un breve período de tiempo
def mostrar_mensaje(pantalla, mensaje):
    mensaje_texto = fuente.render(mensaje, True, BLANCO)
    pantalla.blit(mensaje_texto, (ANCHO // 2 - mensaje_texto.get_width() // 2, ALTO // 2 - mensaje_texto.get_height() // 2))
    pygame.display.flip()  # Actualizamos la pantalla para mostrar el mensaje
    time.sleep(2)  # Esperamos 2 segundos antes de continuar

# Función para cambiar de turno
def cambiar_turno():
    global turno_jugador, juego_terminado, multijugador_terminado, puntuacion_jugador1, puntuacion_jugador2, puntuacion
    if turno_jugador == 1:
        puntuacion_jugador1 = puntuacion
        mostrar_mensaje(pantalla, "¡Turno del Jugador 2!")
        turno_jugador = 2
        reiniciar_juego()
    else:
        puntuacion_jugador2 = puntuacion
        mostrar_mensaje(pantalla, "¡Turno del Jugador 1!")
        juego_terminado = True
        multijugador_terminado = True
        sonido_motor.stop()

# Función para mostrar la pantalla de fin del juego multijugador
def mostrar_pantalla_final_multijugador(pantalla, puntuacion1, puntuacion2):
    pantalla.fill(NEGRO)
    mensaje = fuente_grande.render("Juego Terminado", True, BLANCO)
    pantalla.blit(mensaje, (ANCHO // 2 - mensaje.get_width() // 2, ALTO // 2 - mensaje.get_height() // 2))
    resultado1 = fuente.render(f"Jugador 1: {puntuacion1} puntos", True, BLANCO)
    pantalla.blit(resultado1, (ANCHO // 2 - resultado1.get_width() // 2, ALTO // 2 + 50))
    resultado2 = fuente.render(f"Jugador 2: {puntuacion2} puntos", True, BLANCO)
    pantalla.blit(resultado2, (ANCHO // 2 - resultado2.get_width() // 2, ALTO // 2 + 100))
    if puntuacion1 > puntuacion2:
        mensaje_ganador = fuente_grande.render("¡Jugador 1 Gana!", True, BLANCO)
    elif puntuacion1 < puntuacion2:
        mensaje_ganador = fuente_grande.render("¡Jugador 2 Gana!", True, BLANCO)
    else:
        mensaje_ganador = fuente_grande.render("¡Empate!", True, BLANCO)
    pantalla.blit(mensaje_ganador, (ANCHO // 2 - mensaje_ganador.get_width() // 2, ALTO // 2 + 150))
    boton_inicio_rect = pygame.Rect(ANCHO // 2 - 100, ALTO // 2 + 250, 200, 50)
    pygame.draw.rect(pantalla, GRIS, boton_inicio_rect)
    mensaje_inicio = fuente.render("Inicio", True, BLANCO)
    pantalla.blit(mensaje_inicio, (boton_inicio_rect.x + 60, boton_inicio_rect.y + 10))
    pygame.display.flip()
    return boton_inicio_rect

# Bucle principal del juego
ejecutando = True
juego_iniciado = False

while ejecutando:
    for evento in pygame.event.get():
        if evento.type == pygame.QUIT:
            ejecutando = False
        if evento.type == pygame.MOUSEBUTTONDOWN:
            mouse_x, mouse_y = evento.pos
            if not juego_iniciado:
                if boton_un_jugador_rect.collidepoint(mouse_x, mouse_y):
                    reiniciar_juego()
                    juego_iniciado = True
                    modo_juego = "solo"
                elif boton_multijugador_rect.collidepoint(mouse_x, mouse_y):
                    reiniciar_juego()
                    juego_iniciado = True
                    modo_juego = "multijugador"
                elif boton_salir_rect.collidepoint(mouse_x, mouse_y):
                    ejecutando = False
            else:
                if juego_terminado:
                    if modo_juego == "solo" and boton_inicio_rect.collidepoint(mouse_x, mouse_y):
                        juego_iniciado = False
                        juego_terminado = False
                    elif modo_juego == "multijugador" and boton_inicio_rect.collidepoint(mouse_x, mouse_y):
                        reiniciar_juego()
                        juego_terminado = False
                        multijugador_terminado = False
                        juego_iniciado = False

    if not juego_iniciado:
        boton_un_jugador_rect, boton_multijugador_rect, boton_salir_rect = pantalla_inicio()
    elif not juego_terminado:
        # Actualizar tiempo
        tiempo_restante -= 1 / 30
        if tiempo_restante <= 0:
            if modo_juego == "solo":
                juego_terminado = True
            elif modo_juego == "multijugador" and not multijugador_terminado:
                cambiar_turno()

        # Ajustar velocidad y densidad
        if auto_vel < velocidad_maxima:
            auto_vel += velocidad_incremento / 40
            obstaculo_vel += velocidad_incremento / 30
        if espacio_seguro > espacio_seguro_minimo:
            espacio_seguro -= densidad_incremento / 30
        if densidad < densidad_maxima:
            densidad += densidad_incremento / 30

        # Obtener las teclas presionadas
        # Mover el auto del jugador
        teclas = pygame.key.get_pressed()
        if (teclas[pygame.K_LEFT] or teclas[pygame.K_a]) and auto_x > 0:
            auto_x -= 5
        if (teclas[pygame.K_RIGHT] or teclas[pygame.K_d]) and auto_x < ANCHO - auto_tam[0]:
            auto_x += 5

        # Mover y crear obstáculos
        for obstaculo in obstaculos:
            obstaculo.y += obstaculo_vel
        if len(obstaculos) == 0 or obstaculos[-1].y > espacio_seguro:
            if random.random() < densidad:
                crear_obstaculo()

        # Eliminar obstáculos que salen de la pantalla
        obstaculos = [obstaculo for obstaculo in obstaculos if obstaculo.y < ALTO]

        # Detectar colisión
        auto_rect = pygame.Rect(auto_x, auto_y, auto_tam[0], auto_tam[1])
        colision = any(auto_rect.colliderect(obstaculo) for obstaculo in obstaculos)
        if colision:
            sonido_motor.stop()
            sonido_colision.play()
            if modo_juego == "solo":
                juego_terminado = True
            elif modo_juego == "multijugador" and not multijugador_terminado:
                cambiar_turno()

        # Incrementar puntuación
        puntuacion += 1

        # Dibujar la pantalla
        pantalla.blit(fondo_img, (0, 0))  # Dibujar el fondo
        pantalla.blit(auto_img, (auto_x, auto_y))
        for obstaculo in obstaculos:
            pantalla.blit(obstaculo_img, (obstaculo.x, obstaculo.y))
        mostrar_puntuacion(pantalla, puntuacion)
        mostrar_tiempo(pantalla, tiempo_restante)
        pygame.display.flip()
    else:
        if modo_juego == "solo":
            # Mostrar pantalla de choque
            pantalla.fill(NEGRO)
            mostrar_mensaje_choque(pantalla, puntuacion)
            boton_inicio_rect = pygame.Rect(ANCHO // 2 - 100, ALTO // 2 + 150, 200, 50)
            pygame.draw.rect(pantalla, GRIS, boton_inicio_rect)
            mensaje_inicio = fuente.render("Inicio", True, BLANCO)
            pantalla.blit(mensaje_inicio, (boton_inicio_rect.x + 60, boton_inicio_rect.y + 10))
            pygame.display.flip()
        elif modo_juego == "multijugador":
            # Mostrar pantalla de fin de juego multijugador
            boton_inicio_rect = mostrar_pantalla_final_multijugador(pantalla, puntuacion_jugador1, puntuacion_jugador2)

    # Controlar la velocidad de actualización
    reloj.tick(30)

pygame.quit()

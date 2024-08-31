extends Node2D

@export var enemy_scene: PackedScene  # La escena del enemigo
@export var spawn_interval: float = 1.2  # Intervalo de aparición de enemigos
@export var words: Array = ["comet", "planet", "star", "alien", "spaceship"]  # Palabras para los enemigos

var spawn_timer: float = 1.2  # Temporizador para la aparición de enemigos
var score: int = 0  # Puntaje del jugador
var time_left: float = 60.0  # Tiempo límite en segundos

# Llamado cuando el nodo entra en la escena
func _ready():
	$ScoreLabel.text = "Score: " + str(score)
	$TimeLabel.text = "Time: " + str(time_left)
	# Conectar la señal text_submitted de LineEdit a la función _on_LineEdit_text_submitted
	$LineEdit.connect("text_submitted", Callable(self, "_on_LineEdit_text_submitted"))


# Llamado en cada frame
func _process(delta: float):
	spawn_timer += delta
	if spawn_timer > spawn_interval:
		spawn_enemy()
		spawn_timer = 0.0

	time_left -= delta
	$TimeLabel.text = "Time: " + str(floor(time_left))
	
	if time_left <= 0:
		game_over()

# Genera un nuevo enemigo
func spawn_enemy():
	var enemy = enemy_scene.instantiate()
	var random_word = words[randi() % words.size()]  # Selecciona una palabra aleatoria
	enemy.word = random_word
	enemy.position = $EnemySpawnPoint.position
	add_child(enemy)

# Función llamada cuando se envía texto desde LineEdit
func _on_LineEdit_text_submitted(new_text: String):
	for enemy in get_children():
		if enemy is Area2D and enemy.word == new_text:
			enemy.destroy()
			$LineEdit.clear()
			increment_score()
			break

# Incrementa el puntaje del jugador
func increment_score():
	score += 1
	$ScoreLabel.text = "Score: " + str(score)

# Maneja el final del juego
func game_over():
	get_tree().paused = true
	$LineEdit.disabled = true
	$ScoreLabel.text += " - Game Over!"

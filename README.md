import RPi.GPIO as GPIO
import time

# Nastavení GPIO pinů
RED_PIN = 17   # GPIO pin pro červenou
GREEN_PIN = 27 # GPIO pin pro zelenou
BLUE_PIN = 22  # GPIO pin pro modrou
ENCODER_PIN_A = 23  # GPIO pin pro otočný enkodér (kanál A)
ENCODER_PIN_B = 24  # GPIO pin pro otočný enkodér (kanál B)
BUTTON_PIN = 25     # GPIO pin pro tlačítko

# Nastavení pro PWM frekvenci (bez blikání)
PWM_FREQ = 1000

# Počáteční stav
current_color = [0, 0, 0]  # Červená, zelená, modrá složka
selected_color = 0  # Vybraná složka (0: červená, 1: zelená, 2: modrá)
brightness_step = 10  # Krok pro jas

def setup():
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(RED_PIN, GPIO.OUT)
    GPIO.setup(GREEN_PIN, GPIO.OUT)
    GPIO.setup(BLUE_PIN, GPIO.OUT)
    GPIO.setup(ENCODER_PIN_A, GPIO.IN, pull_up_down=GPIO.PUD_UP)
    GPIO.setup(ENCODER_PIN_B, GPIO.IN, pull_up_down=GPIO.PUD_UP)
    GPIO.setup(BUTTON_PIN, GPIO.IN, pull_up_down=GPIO.PUD_UP)

    # PWM pro LED
    global red_pwm, green_pwm, blue_pwm
    red_pwm = GPIO.PWM(RED_PIN, PWM_FREQ)
    green_pwm = GPIO.PWM(GREEN_PIN, PWM_FREQ)
    blue_pwm = GPIO.PWM(BLUE_PIN, PWM_FREQ)
    red_pwm.start(0)
    green_pwm.start(0)
    blue_pwm.start(0)

def update_led():
    """Aktualizace barev LED podle aktuálního stavu"""
    red_pwm.ChangeDutyCycle(current_color[0])
    green_pwm.ChangeDutyCycle(current_color[1])
    blue_pwm.ChangeDutyCycle(current_color[2])
    print(f"Aktuální barva: R={current_color[0]}, G={current_color[1]}, B={current_color[2]}")

def encoder_callback(channel):
    """Zpracování otáčení enkodéru"""
    global current_color
    a_state = GPIO.input(ENCODER_PIN_A)
    b_state = GPIO.input(ENCODER_PIN_B)
    
    # Otočení doprava
    if a_state == b_state:
        current_color[selected_color] = min(current_color[selected_color] + brightness_step, 100)
    else:
        current_color[selected_color] = max(current_color[selected_color] - brightness_step, 0)
    
    update_led()

def button_callback(channel):
    """Přepínání mezi barvami po stisknutí tlačítka"""
    global selected_color
    selected_color = (selected_color + 1) % 3  # Cyklování mezi červenou, zelenou a modrou
    print(f"Vybraná barva: {'Červená' if selected_color == 0 else 'Zelená' if selected_color == 1 else 'Modrá'}")

def loop():
    """Hlavní smyčka programu"""
    GPIO.add_event_detect(ENCODER_PIN_A, GPIO.BOTH, callback=encoder_callback)
    GPIO.add_event_detect(BUTTON_PIN, GPIO.FALLING, callback=button_callback, bouncetime=300)
    
    try:
        while True:
            time.sleep(0.1)
    except KeyboardInterrupt:
        pass
    finally:
        GPIO.cleanup()

if __name__ == '__main__':
    setup()
    loop()

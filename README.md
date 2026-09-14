import os
from PIL import Image, ImageFilter, ImageDraw, ImageFont
from moviepy import ImageClip, concatenate_videoclips

def preparar_imagen_vertical(ruta_imagen, archivo_salida_temp):
    """Toma la imagen, la convierte a RGB, le pone fondo borroso, marca arriba y contactos abajo."""
    img = Image.open(ruta_imagen).convert("RGB")
    
    # 1. Fondo borroso para llenar la pantalla vertical (1080x1920)
    ratio_fondo = max(1080 / img.width, 1920 / img.height)
    fondo_w = int(img.width * ratio_fondo)
    fondo_h = int(img.height * ratio_fondo)
    fondo = img.resize((fondo_w, fondo_h)).crop((0, 0, 1080, 1920))
    fondo = fondo.filter(ImageFilter.GaussianBlur(40)).point(lambda p: p * 0.4) # Oscurecido
    
    # 2. Imagen nítida en el centro (dejando espacio arriba para el logo y abajo para contactos)
    ratio_img = 850 / img.width
    img_h = int(img.height * ratio_img)
    img_nitida = img.resize((850, img_h))
    
    # Centrar la imagen verticalmente entre las franjas
    pos_y = (1920 - img_h) // 2
    fondo.paste(img_nitida, ((1080 - 850) // 2, pos_y))
    
    # 3. Dibujar textos con PIL (Diseño Gráfico Automatizado)
    draw = ImageDraw.Draw(fondo)
    
    try:
        fuente_titulo = ImageFont.truetype("arialbd.ttf", 65)
        fuente_contacto = ImageFont.truetype("arialbd.ttf", 45)
        fuente_web = ImageFont.truetype("arial.ttf", 40)
    except:
        fuente_titulo = ImageFont.load_default()
        fuente_contacto = ImageFont.load_default()
        fuente_web = ImageFont.load_default()
        
    # --- FRANJA SUPERIOR (MARCA) ---
    draw.rectangle([(0, 80), (1080, 220)], fill=(34, 139, 34)) # Verde Plastiagro
    draw.text((540, 150), "PLASTIAGRO - MACIZOS", fill="white", font=fuente_titulo, anchor="mm")
    
    # --- FRANJA INFERIOR (CONTACTOS CLAVE) ---
    draw.rectangle([(0, 1680), (1080, 1880)], fill=(20, 20, 20)) # Fondo oscuro elegante
    draw.text((540, 1735), "WhatsApp: 319 797 7572", fill=(255, 204, 0), font=fuente_contacto, anchor="mm") # Amarillo llamativo
    draw.text((540, 1815), "Web: plastiagro.42web.io", fill="white", font=fuente_web, anchor="mm")
    
    # Guardamos el frame listo
    fondo.save(archivo_salida_temp)

def crear_carrusel_tiktok(lista_imagenes, archivo_salida):
    print("1. Procesando imágenes con diseño y contactos de Plastiagro...")
    clips = []
    archivos_temp = []
    
    # Cada foto durará 2.5 segundos en pantalla para dar tiempo a leer el contacto
    for i, ruta in enumerate(lista_imagenes):
        if os.path.exists(ruta):
            temp_name = f"temp_{i}.jpg"
            preparar_imagen_vertical(ruta, temp_name)
            
            clip = ImageClip(temp_name, duration=2.5)
            clips.append(clip)
            archivos_temp.append(temp_name)
        else:
            print(f"[!] No se encontró la imagen: {ruta}")
            
    if not clips:
        print("No hay imágenes válidas para procesar.")
        return

    print("2. Uniendo las escenas en el video final...")
    video_final = concatenate_videoclips(clips, method="compose")
    
    print("3. Exportando video MP4 profesional...")
    video_final.write_videofile(archivo_salida, fps=24, codec='libx264')
    
    # Limpiar archivos temporales
    for temp in archivos_temp:
        os.path.exists(temp) and os.remove(temp)
        
    print(f"\n¡Video Pro creado con éxito! Revisa tu archivo: {archivo_salida}")

# Tus tres imágenes listas en la carpeta
fotos_catalogo = [
    "producto.JPG", 
    "3.png", 
    "4.png"
] 

# Ejecutamos la orden
crear_carrusel_tiktok(fotos_catalogo, "carrusel_plastiagro_pro.mp4")

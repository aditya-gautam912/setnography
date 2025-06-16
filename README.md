from PIL import Image

# Converts a string into its binary representation
def to_bin(data):
    return ''.join(format(ord(i), '08b') for i in data)

# Converts binary back into string
def from_bin(bin_data):
    chars = [chr(int(bin_data[i:i+8], 2)) for i in range(0, len(bin_data), 8)]
    return ''.join(chars)

# Encodes a message into the image
def encode(image_path, message, output_path):
    img = Image.open(image_path)
    binary = to_bin(message + '###')  # Delimiter to mark end of message
    data_index = 0
    img_data = img.getdata()
    
    new_data = []
    for item in img_data:
        if data_index < len(binary):
            r = item[0] & ~1 | int(binary[data_index])
            data_index += 1
        else:
            r = item[0]

        if data_index < len(binary):
            g = item[1] & ~1 | int(binary[data_index])
            data_index += 1
        else:
            g = item[1]

        if data_index < len(binary):
            b = item[2] & ~1 | int(binary[data_index])
            data_index += 1
        else:
            b = item[2]

        new_data.append((r, g, b))
    
    img.putdata(new_data)
    img.save(output_path)
    print("✅ Message encoded and saved to:", output_path)

# Decodes the hidden message from the image
def decode(image_path):
    img = Image.open(image_path)
    binary = ''
    for pixel in img.getdata():
        for value in pixel[:3]:
            binary += str(value & 1)
    
    all_bytes = [binary[i:i+8] for i in range(0, len(binary), 8)]
    decoded = ''
    for byte in all_bytes:
        char = chr(int(byte, 2))
        decoded += char
        if decoded.endswith('###'):
            break
    return decoded[:-3]

# ------------------------------------------
# ✅ Example usage
# ------------------------------------------

# Provide valid .png files here
input_image = r"C:\Users\ag950\Downloads\VeraCrypt\docs\html\en\ru\flag-nz.png"  # Original image
output_image = r"D:\stego_image_xor.png"  # Output image

# Save a sample image if you don't have one
# You can create a blank image for testing
def create_sample_image(path):
    img = Image.new('RGB', (300, 300), color='white')
    img.save(path)
    print("Sample image created at:", path)

# Create sample image if it doesn't exist
import os
if not os.path.exists(input_image):
    create_sample_image(input_image)

# The secret message to hide
secret_message = "This is a hidden message!"

# Encode and decode the message
encode(input_image, secret_message, output_image)
print("🔍 Decoded message:", decode(output_image))

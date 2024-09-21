from flask import Flask, request, send_file, render_template, jsonify
import os
import subprocess
import uuid
import magic
import threading
import re
import time

app = Flask(__name__)
app.config['TEMPLATES_AUTO_RELOAD'] = True

UPLOAD_FOLDER = 'uploads'
PROCESSED_FOLDER = 'processed'
ALLOWED_EXTENSIONS = {'mp4', 'avi', 'mkv', 'mov'}

if not os.path.exists(UPLOAD_FOLDER):
    os.makedirs(UPLOAD_FOLDER)
if not os.path.exists(PROCESSED_FOLDER):
    os.makedirs(PROCESSED_FOLDER)

# Global dictionary to store progress
progress_dict = {}

def allowed_file(filename):
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

def check_file_type(file_path):
    mime = magic.Magic(mime=True)
    file_type = mime.from_file(file_path)
    return file_type.startswith('video/')

def get_video_duration(filepath):
    cmd = ['ffprobe', '-v', 'error', '-show_entries', 'format=duration', '-of', 'default=noprint_wrappers=1:nokey=1', filepath]
    result = subprocess.run(cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
    return float(result.stdout)

def process_video(input_path, output_path, action, duration, task_id, original_filename):
    global progress_dict
    progress_dict[task_id] = {'progress': 0, 'filename': ''}

    if action == 'compress':
        command = [
            'ffmpeg',
            '-i', input_path,
            '-vf', 'scale=1920:-1',
            '-c:v', 'libx264',
            '-crf', '23',
            '-preset', 'medium',
            '-c:a', 'aac',
            '-b:a', '128k',
            '-progress', 'pipe:1',
            output_path
        ]
        suffix = 'compressed'
    else:  # remux
        command = [
            'ffmpeg',
            '-i', input_path,
            '-c', 'copy',
            '-progress', 'pipe:1',
            output_path
        ]
        suffix = 'remuxed'

    process = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, universal_newlines=True)
    
    for line in process.stdout:
        progress_match = re.search(r'out_time_ms=(\d+)', line)
        if progress_match:
            time_in_ms = int(progress_match.group(1))
            progress = min(99, int((time_in_ms / 1000000) / duration * 100))
            progress_dict[task_id]['progress'] = progress

    process.wait()
    
    if process.returncode == 0:
        progress_dict[task_id]['progress'] = 100
        name, ext = os.path.splitext(original_filename)
        new_filename = f"{name}_{suffix}{ext}"
        os.rename(output_path, os.path.join(PROCESSED_FOLDER, new_filename))
        progress_dict[task_id]['filename'] = new_filename
    else:
        progress_dict[task_id]['progress'] = -1  # Indicate an error

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return jsonify({'error': 'No file part'}), 400
    file = request.files['file']
    if file.filename == '':
        return jsonify({'error': 'No selected file'}), 400
    if file and allowed_file(file.filename):
        filename = str(uuid.uuid4()) + os.path.splitext(file.filename)[1]
        filepath = os.path.join(UPLOAD_FOLDER, filename)
        file.save(filepath)
        
        if not check_file_type(filepath):
            os.remove(filepath)
            return jsonify({'error': 'File is not a valid video'}), 400
        
        output_filename = filename  # Use the UUID filename for processing
        output_filepath = os.path.join(PROCESSED_FOLDER, output_filename)
        
        action = request.form.get('action', 'remux')
        task_id = str(uuid.uuid4())
        
        try:
            duration = get_video_duration(filepath)
            thread = threading.Thread(target=process_video, args=(filepath, output_filepath, action, duration, task_id, file.filename))
            thread.start()
            
            return jsonify({'task_id': task_id}), 200
        except Exception as e:
            app.logger.error(f"Processing error: {str(e)}")
            return jsonify({'error': f'Processing error: {str(e)}'}), 500

@app.route('/progress/<task_id>')
def get_progress(task_id):
    task_info = progress_dict.get(task_id, {'progress': 0, 'filename': ''})
    return jsonify(task_info)

@app.route('/processed/<filename>')
def download_file(filename):
    return send_file(os.path.join(PROCESSED_FOLDER, filename), as_attachment=True)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5333)

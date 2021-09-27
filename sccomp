#!/usr/bin/env python3
# pyright: reportMissingImports=false

import os
import subprocess
import sys
import shutil
import argparse
from pathlib import Path
import re

import pypandoc

HTML_HEADER = '''<!DOCTYPE html>
<html>
<head>
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />

<style>
img {
    width: 100%;
}

h5 {
	font-size: 2.5rem;
	width: 100%;
	text-align: center;
}

p {
	font-size: 1rem;
}

h1 {
	font-size: 2.5rem;
}

h2 {
	font-size: 2rem;
}

h3 {
	font-size: 1.75rem;
}

h4 {
	font-size: 1.5rem;
}

* {
  font-family: Arial, Helvetica, sans-serif;
}

a {
	color: #0554ff;
	text-decoration: none;
}

body {
	padding-bottom: 32px;
	margin: 0;
}

:root {
	--light-button: #eee;
	--light-button-text: #000;
	--reading-button: #e5decc;
	--reading-button-text: #33271e;
	--dark-button: #222;
	--dark-button-text: #555;

	--button-light: #eba134;
	--button-reading: #ff6a00;
	--button-dark: #fff;
}

a:visited {
	color: #084596;
}

.container {
  padding-right: 15px;
  padding-left: 15px;
  margin-right: auto;
  margin-left: auto;
}

@media (min-width: 600px) {
  .container {
    width: 600px;
  }
}
/* Themes */

.reading {
	/*color: #5b4636;*/
	color: #33271e;
	background: #f4ecd8;
}

.dark {
	color: white;
	background: #111;
}

/* Theme buttons */

.switch-toggle {
   display: flex;
   border: solid 1px #000;
   border-top: none;
   width: max-content;
   margin: auto;
   border-radius: 0 0 10px 10px;
}

body.light > div > div {
	border-color: #000;
	background: var(--light-button, #eee);
}
body.light > div > div input + label {
	color: var(--light-button-text);
}

body.reading > div > div {
	border-color: var(--reading-button-text);
	background: var(--reading-button);
}
body.reading > div > div input + label {
	color: var(--reading-button-text);
}

body.dark > div > div {
	border-color: #fff;
	background: var(--dark-button);
}
body.dark > div > div input + label {
	color: var(--dark-button-text);
}


.switch-toggle input {
  position: absolute;
  opacity: 0;
  top: -200px;
}

.switch-toggle input + label {
  padding: 7px;
  color: var(--button-text);
  cursor: pointer;
}

.switch-toggle input:checked + label#light {
  color: var(--button-light);
}

.switch-toggle input:checked + label#reading {
  color: var(--button-reading);
}

.switch-toggle input:checked + label#dark {
  color: var(--button-dark);
}

</style>

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.1.1/css/all.min.css" crossorigin="anonymous">

</head>
</body>

<div class="container">

<div class="theme_container switch-toggle">

<input id="light" name="state-d" type="radio" checked>
<label for="light" id="light" onclick=""><i class="fas fa-sun"></i></label>

<input id="reading" name="state-d" type="radio">
<label for="reading" id="reading" onclick=""><i class="fas fa-book"></i></label>

<input id="dark" name="state-d" type="radio">
<label for="dark" id="dark" onclick=""><i class="fas fa-moon"></i></label>

</div>'''

HTML_FOOTER = '''
</div>

<script>

const radios = document.getElementsByTagName("input");

radios[0].onclick = () => {
	document.body.classList = [];
};

radios[1].onclick = () => {
	document.body.classList = ['reading'];
};

radios[2].onclick = () => {
	document.body.classList = ['dark'];
};

</script>

	</body>
</html>'''

DATABASE_FILE='.dbfiles'

def CheckIfFileIsModified(filenamee):
    '''
    Check if file is modified from database
    '''

    def WriteRows(rowdata):
        with open(DATABASE_FILE, 'w') as database_file_writable:
            final = ''
            for current_row in rowdata:
                final += f'{current_row[0]},{current_row[1]}\n'
            database_file_writable.write(final)
            database_file_writable.close()

    if not os.path.isfile(DATABASE_FILE):
        with open(DATABASE_FILE, 'w+') as j:
            j.close()

    index = -1
    with open(DATABASE_FILE, 'r') as database_file:
        data = database_file.read().split('\n')[:-1]
        csvreader = [i.split(',') for i in data]
        rows = []
        for i, row in enumerate(csvreader):
            rows.append(row)
            if row[0] == filenamee:
                index = i
    modification_time = os.path.getmtime(args.source + '/' + filenamee)

    if index == -1:
        index = len(rows)
        rows.append([filenamee, modification_time])
        WriteRows(rows)
        return True

    if float(rows[index][1]) >= modification_time:
        if not os.path.isfile('Build/' + rows[index][0]):
            rows.pop(index)
            WriteRows(rows)
        return False
    rows[index][1] = modification_time
    WriteRows(rows)
    return True

def LReplace(pattern, sub, string):
    """
    Replaces 'pattern' in 'string' with 'sub' if 'pattern' starts 'string'.
    """
    return re.sub(f'^{pattern}', sub, string)

# https://stackoverflow.com/a/47655463
def GetFiles(extensions):
    ''' This function gets all files recursively with the specified extensions.'''
    all_files = []
    for ext in extensions:
        all_files.extend(Path(args.source).rglob(ext))
    return all_files

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('-s', '--source', help='Source directory', type=str, default='src')
    parser.add_argument('-o', '--output', help='Build directory', type=str, default='Build')
    parser.add_argument('--pdf', help='Output pdf data', action='store_true')
    parser.add_argument('--all', help='Output pdf data', action='store_true')
    args = parser.parse_args()

    if not os.path.isdir(args.source):
        print(f"Source directory `{args.source}` does not exist or is a file.")
        sys.exit(1)

    source_files = GetFiles(('*.txt', '*.md', '*.png', '*.jpg', '*.jpeg', '*.c', '*.cpp', '*.sh', '*.in', 'Makefile'))
    for file in source_files:
        file_path = LReplace(args.source + '/', '', str(file))
        filename, file_extension = os.path.splitext(file_path)

        if args.all or CheckIfFileIsModified(file_path):
            if file_extension in ('.txt', '.md'):
                print(f'Compiling `{file_path}`')
                file_parent = args.output + '/' + os.sep.join(filename.split(os.sep)[:-1])
                OUTPUT_FORMAT = 'pdf' if args.pdf else 'html'
                output_file = file_parent + '/' + os.path.basename(filename) + '.' + OUTPUT_FORMAT
                os.makedirs(file_parent, exist_ok=True)

                if not args.pdf:
                    output = HTML_HEADER + pypandoc.convert_file(str(file), OUTPUT_FORMAT, format='md') + HTML_FOOTER
                    with open(output_file, 'w+') as f:
                        f.write(output)
                        f.close()
                else:
                    pypandoc.convert_file(str(file), 'pdf', format='md', outputfile=output_file,
                        extra_args=['-V', 'geometry:margin=1in', f'--resource-path={file_parent}'])
            elif file_extension in ('.png', '.jpg', '.jpeg', '.sh', '.in') or filename == 'Makefile':
                print(f'Copying asset `{file_path}`')
                file_parent = args.output + '/' + os.sep.join(filename.split(os.sep)[:-1])
                output_file = file_parent + '/' + os.path.basename(filename) + file_extension
                os.makedirs(file_parent, exist_ok=True)
                shutil.copy(str(file), output_file)
            elif file_extension == '.cpp':
                print(f'Compiling C++ file `{file_path}`')
                file_parent = args.output + '/' + os.sep.join(filename.split(os.sep)[:-1])
                output_file = file_parent + '/' + os.path.basename(filename) + '.o'
                os.makedirs(file_parent, exist_ok=True)
                with subprocess.Popen(['g++', '-Wall', '-std=c++20', '-o', output_file, str(file)]) as s:
                    s.wait()
            elif file_extension == '.c':
                print(f'Compiling C++ file `{file_path}`')
                file_parent = args.output + '/' + os.sep.join(filename.split(os.sep)[:-1])
                output_file = file_parent + '/' + os.path.basename(filename) + '.o'
                os.makedirs(file_parent, exist_ok=True)
                with subprocess.Popen(['gcc', '-Wall', '-std=c11', '-o', output_file, str(file)]) as s:
                    s.wait()

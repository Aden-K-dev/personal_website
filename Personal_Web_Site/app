import os
import json
import pandas as pd
import matplotlib.pyplot as plt
import plotly.express as px
import plotly
from flask import Flask, render_template, request, redirect, url_for, session
from werkzeug.utils import secure_filename
from flask_admin import Admin, AdminIndexView
from flask_admin.contrib.fileadmin import FileAdmin
from flask_admin.contrib.sqla import ModelView

app = Flask(__name__)

app.secret_key = 'no_no_no'

UPLOAD_FOLDER = 'static/uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
os.makedirs(UPLOAD_FOLDER, exist_ok=True)


GRAPH_FOLDER = 'static/images'
os.makedirs(GRAPH_FOLDER, exist_ok=True)

class MyAdminIndexView(AdminIndexView):
    def is_accessible(self):
        return session.get('admin_logged_in')

    def inaccessible_callback(self, name, **kwargs):
        return redirect(url_for('admin_login'))

# Initialize Flask-Admin with default templates
admin = Admin(app, name='Admin Panel', template_mode='bootstrap4', index_view=MyAdminIndexView())

# Add views for managing uploads and images
admin.add_view(FileAdmin(app.config['UPLOAD_FOLDER'], '/uploads/', name='Uploads', endpoint='uploads_admin'))
admin.add_view(FileAdmin(GRAPH_FOLDER, '/images/', name='Images', endpoint='images_admin'))

from flask_admin.base import BaseView, expose

class BlogPostAdminView(BaseView):
    @expose('/')
    def index(self):
        try:
            with open('blog_posts.json', 'r') as f:
                blog_posts = json.load(f)
        except FileNotFoundError:
            blog_posts = []

        return self.render('admin/blog_posts.html', blog_posts=blog_posts)

    @expose('/delete/<int:post_id>')
    def delete(self, post_id):
        try:
            with open('blog_posts.json', 'r') as f:
                blog_posts = json.load(f)
        except FileNotFoundError:
            blog_posts = []

        if 0 <= post_id < len(blog_posts):
            blog_posts.pop(post_id)

        with open('blog_posts.json', 'w') as f:
            json.dump(blog_posts, f)

        return redirect(url_for('.index'))

admin.add_view(BlogPostAdminView(name='Blog Posts', endpoint='blogpostadminview'))

try:
    with open('blog_posts.json', 'r') as f:
        blog_posts = json.load(f)
except FileNotFoundError:
    blog_posts = []

questions = []


@app.route('/')
def main():
    return render_template('main.html')


@app.route('/graph')
def graph():
    csv_file = 'c:\\Users\\kuhn512cs12\\Downloads\\4016573.csv'

    try:
        data = pd.read_csv(csv_file, header=0)
        data['DATE'] = pd.to_datetime(data['DATE'], format='%Y-%m-%d', errors='coerce')
        data['TMAX'] = pd.to_numeric(data['TMAX'], errors='coerce')
        data['TMIN'] = pd.to_numeric(data['TMIN'], errors='coerce')
        data = data.dropna(subset=['DATE', 'TMAX', 'TMIN'])

        data['DATE'] = data['DATE'].dt.strftime('%Y-%m-%d')

        trace_tmax = {
            'x': data['DATE'].tolist(),
            'y': data['TMAX'].tolist(),
            'type': 'scatter',
            'name': 'TMAX',
            'line': {'color': 'red'}
        }
        trace_tmin = {
            'x': data['DATE'].tolist(),
            'y': data['TMIN'].tolist(),
            'type': 'scatter',
            'name': 'TMIN',
            'line': {'color': 'blue'}
        }

        graph_data = [trace_tmax, trace_tmin]

        graph_json = json.dumps(graph_data)
        return render_template('graph.html', graph_json=graph_json)

    except Exception as e:
        return f"An error occurred while processing the CSV file: {e}"


@app.route('/blog', methods=['GET', 'POST'])
def blog():
    if request.method == 'POST':
        title = request.form['title']
        content = request.form['content']
        image = request.files['image']

        image_filename = None
        if image and image.filename != '':
            image_filename = secure_filename(image.filename)
            image.save(os.path.join(app.config['UPLOAD_FOLDER'], image_filename))

        new_post = {
            "title": title,
            "content": content,
            "image": image_filename
        }
        blog_posts.append(new_post)

        with open('blog_posts.json', 'w') as f:
            json.dump(blog_posts, f)

        return redirect(url_for('blog'))

    return render_template('blog.html', blog_posts=blog_posts)


@app.route('/q_and_a', methods=['GET', 'POST'])
def q_and_a():
    admin_password = "no" #should you do passwords like this? no, but its a proof of concept and why do you want to answer random questions anyway? this lets you test it out

    if request.method == 'POST':
        if 'question' in request.form:
            question = request.form['question']
            questions.append({"question": question, "answer": None})
        elif 'answer' in request.form:
            if request.form.get('password') == admin_password:
                question_index = int(request.form['question_index'])
                answer = request.form['answer']
                if 0 <= question_index < len(questions):
                    questions[question_index]['answer'] = answer
            else:
                return "Incorrect admin password", 403
        return redirect(url_for('q_and_a'))
    return render_template('q_and_a.html', questions=questions, enumerate=enumerate)


@app.route('/contact', methods=['GET', 'POST'])
def contact():
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        message = request.form['message']

        contact_data = {"name": name, "email": email, "message": message}
        with open('contact_messages.json', 'a') as f:
            f.write(json.dumps(contact_data) + '\n')

        return render_template('contact_confirmation.html')

    return render_template('contact.html')


@app.route('/admin_login', methods=['GET', 'POST'])
def admin_login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        if username == 'admin' and password == 'no': 
            session['admin_logged_in'] = True
            return redirect(url_for('admin_panel'))
        else:
            return "Invalid credentials", 403
    return render_template('admin/admin_login.html')


@app.route('/admin_panel')
def admin_panel():
    if not session.get('admin_logged_in'):
        return redirect(url_for('admin_login'))
    return render_template('admin/admin_panel.html')


@app.route('/admin_logout')
def admin_logout():
    session.pop('admin_logged_in', None)
    return redirect(url_for('main'))


if __name__ == '__main__':
    app.run(debug=True)
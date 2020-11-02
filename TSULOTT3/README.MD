# TSULOTT3
Source code:
```
from flask import Flask, session, request, render_template, render_template_string
from flask_session import Session
from random import randint as ri

app = Flask(__name__)
SESSION_TYPE = 'filesystem'
app.config.from_object(__name__)
Session(app)
cheat = "Pls Don't cheat! "

def check_session(input):
	if session.get(input) == None:
		return ""
	return session.get(input)

@app.route("/", methods=["GET","POST"])
def index():
	try:
		session.pop("name")
		session.pop("jackpot")
	except:
		pass
	if request.method == "POST":
		ok = request.form['ok']
		session["name"] = request.form['name']
		if ok == "Go":
			session["check"] = "access"
			jackpot = " ".join(str(x) for x in [ri(10,99), ri(10,99), ri(10,99), ri(10,99), ri(10,99), ri(10,99)]).strip()
			session["jackpot"] = jackpot
			return render_template_string("Generating jackpot...<script>setInterval(function(){ window.location='/guess'; }, 500);</script>")
	return render_template("start.html")

@app.route('/guess', methods=["GET","POST"])
def guess():
	try:
		if check_session("check") == "":
			return render_template_string(cheat+check_session("name"))
		else:
			if request.method == "POST":
				jackpot_input = request.form['jackpot']
				if jackpot_input == check_session("jackpot"):
					mess = "Really? GG "+check_session("name")+", here your flag: ASCIS{xxxxxxxxxxxxxxxxxxxxxxxxx}"
				elif jackpot_input != check_session("jackpot"):
					mess = "May the Luck be with you next time!<script>setInterval(function(){ window.location='/reset_access'; }, 1200);</script>"
				return render_template_string(mess)
			return render_template("guess.html")
	except:
		pass
	return render_template_string(cheat+check_session("name"))


@app.route('/reset_access')
def reset():
	try:
		session.pop("check")
		return render_template_string("Reseting...<script>setInterval(function(){ window.location='/'; }, 500);</script>")
	except:
		pass
	return render_template_string(cheat+check_session("name"))


if __name__ == "__main__":
	app.secret_key = 'xxxxxxxxxxxxxxxxxxxxx'
	app.run()
```
## The challenge flow: 
At "/" endpoint, your session name and jackpot get pop if exists, and it will add your form name into session "name" + gen new "jackpot" number.

At "/guess" endpoint, it will check if your session "check" value equals "" (empty string) or not, if not it will let you pass or else it will render template string with your entered username.

After the "/guess" part, if your guess is wrong it will redirect you to "/reset_access" endpoint. 

At "/reset_access" endpoint, it will try to pop the "check" in session. If it failed, you will be greeted with the same template string with your "name" value in session.

## The bug
There are 2 bug in this challenge: The clumsy pop and the render_template_string. Lets go one by one.

The first one is the one my teammate found during contest. As you can see at the "/" endpoint it pop "username" and "jackpot" but not "check" so you can enter your username and get to the "/guess" endpoint, connect to the "/" endpoint and then submit "" (empty string) to the server. Because you connected to "/" after you enter "/guess" endpoint, your session "jackpot" value got popped -> empty string. Ez flag.

The second one is the intended solution, which is SSTI. you can see the username get rendered without check, so if you enter `{1+1}` it will return 2. Ez RCE.

`FLAG : ASCIS{...something...SSTI...}` 

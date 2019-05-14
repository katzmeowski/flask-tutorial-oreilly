# flask-tutorial-oreilly
Building the tutorial app from Miguel Grinberg's "Flask Web Development," 2nd ed., published by O'Reilly

## Development

In order to continue working on this project, you will need to set up a virtual environment with the proper dependencies and versions. The following Linux commands will suffice:

```sh
# Clone the project repo
# In project folder, create a new virtualenv called env
# Naming it env will ensure it is ignored by git
$ python -m venv env

# Activate the virtualenv
$ source env/bin/activate

# Install required dependencies with pip
$ pip install -r requirements.txt

# If dependencies or versions need to be changed, update requirements.txt
$ pip freeze > requirements.txt

# All development work must be done while the virtualenv is activated. Running the app will fail if attempted without virtualenv.

# Deactivate the virtualenv if needed
$ deactivate
```

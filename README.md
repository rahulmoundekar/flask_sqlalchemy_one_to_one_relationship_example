# Flask CURD SqlAlchemy one to one relationship example :

#### Project Setup

  - Making the project as :
     ```
        mkdir flask_sqlalchemy_one_to_one_relationship_example
		cd flask_sqlalchemy_one_to_one_relationship_example
    ```
  - Install flask:
    ```
        pip install flask
    ```
 - Integrating SqlAlchemy
    ```
      pip install sqlalchemy
    ```
 - Create EmployeeManagementSystem.py for development    
  - Declaring Models:
     ```
    class Person(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), nullable=False)
    mobile = db.Column(db.String(12), nullable=False)
    license = db.relationship('License', backref=db.backref('person', uselist=False), cascade='all, delete-orphan',
                              lazy=True,
                              uselist=False)  # delete parent record in one to many relationship in flask cascade='all, delete-orphan'

    def __repr__(self):
        return f'Person: {self.id}, {self.name}, {self.mobile}'


    class License(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        license_number = db.Column(db.String(80), nullable=False)
        issue_date = db.Column(db.DateTime, nullable=False)
        expiry_date = db.Column(db.DateTime, nullable=False)
        person_id = db.Column(db.Integer, db.ForeignKey('person.id'), unique=True, nullable=False)
    
        def __repr__(self):
            return f'License: {self.id}, {self.license_number}, {self.issue_date}, {self.expiry_date}'

    ```
 - create settings.py for configuration
     ```
     # configuration
    class Config:
        DEBUG = True
        # db
        SQLALCHEMY_DATABASE_URI = 'mysql://root:root@localhost/djangoapp'
        SQLALCHEMY_TRACK_MODIFICATIONS = False
    ```
    
 - Make a runserver configuration
     ``` 
    app = Flask(__name__)
    app.config.from_object('settings.Config')
    db = SQLAlchemy(app)
    
    if __name__ == "__main__":
        app.run(debug=True)
    ```
 - create html file inside templates folder
    * check project directory for index.html file
    
 - create curd def in EmployeeManagementSystem.py
    ``` 
      @app.route("/", methods=['GET', 'POST'])
         @app.route("/<int:person_id>", methods=['GET'])
        def PersonHome(person_id=None):
            if request.method == 'POST':
                # person data
                pid = request.form.get('id')
                name = request.form.get('name')
                mobile = request.form.get('mobile')
                # license data
                license_number = request.form.get('license_number')
                issue_date = request.form.get('issue_date')
                expiry_date = request.form.get('expiry_date')
        
                if pid:  # if id present update records
                    person = Person.query.filter_by(id=pid).first()
                    person.name = name
                    person.mobile = mobile
        
                    person.license.license_number = license_number
                    person.license.issue_date = issue_date
                person.license.expiry_date = expiry_date
    
                db.session.commit()
            else:
                # id not None save record
                try:
                    person_entry = Person(name=name, mobile=mobile)
                    license_entry = License(license_number=license_number, issue_date=issue_date, expiry_date=expiry_date)
                    person_entry.license = license_entry
                    db.session.add(person_entry)
                    db.session.commit()
                except IntegrityError as e:
                    print(e, 'Something went wrong please try again later')
        person = None
        licenses = None
        if person_id:  # load record form edit form data
            person = Person.query.filter_by(id=person_id).first()
            licenses = License.query.filter_by(person_id=person_id).first()
        persons = Person.query.all()
    
        return render_template('index.html', persons=persons, person=person, license=licenses)


    @app.route("/delete/<int:person_id>", methods=['GET'])
    def deletePerson(person_id):
        try:
            person = Person.query.filter_by(id=person_id).first()
            print(person)
            if person:
                db.session.delete(person)
                db.session.commit()
            else:
                print('Could not find any note to delete')
        except IntegrityError as e:
            print(e, 'Something went wrong please try again later')
        return redirect("/")

      ``` 
 - In order to run app:
      ```
	python PersonLicenseManagement.py
      ```

 - run on your browser
    * Your should run at: http://127.0.0.1:5000/

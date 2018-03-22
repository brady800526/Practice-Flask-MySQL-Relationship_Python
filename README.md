## Summary

This repo is a practice of using Python Framework Flask-SQLAlchemy.
In this practice, we use Flask-SQLAlchemy to build one-to-many relationship and many-to-many relationship.

*All the steps are followed by the [Pretty Printed Website](https://prettyprinted.com/courses/enrolled/168305).*

## Usage

Download the repo, and get into the Python Shell under this directory:

    from application import db, Member, Order, Course
    db.create_all()

This step will create db and build the User, Order and Course Model.

#### One-to-Many Relationship

Each Member will have many Orders. To create user and connects with orders and commit to the database(db):

    Brady = Member(username="Brady", password="password", email="brady@gmail.com", join_date=datetime.datetime.now())

    db.session.add(Brady)
    db.session.commit()

    order1 = Order(price=50, member_id=Brady.id)
    order2 = Order(price=100, member_id=Brady.id)

    db.session.add(order1)
    db.session.add(order2)
    db.session.commit()

To get the orders from user:

    Brady = Member.query.filter(Member.username == "Brady").all()
    orders = Brady.orders.all()

<figure style="text-align: center;">
    <img src="README_img/Relationship.001.jpeg" alt="One-to-Many Relationship" style="width: 80%; height: 80%"/>
    <figcaption style="display: block;">One-to-Many Relationship</figcaption>
</figure>

#### Many-to-Many Relationship

Each member have taken some courses, but also, each course can have multiple members. To create member connect with courses, we can type:
    
    Brady = Member(username="Brady", password="password", email="brady@gmail.com", join_date=datetime.datetime.now())
    Paul = Member(username="Paul", password="password", email="paul@gmail.com", join_date=datetime.datetime.now())

    db.session.add(Brady)
    db.session.add(Paul)
    db.session.commit()

    course1 = Course(name='Course One')
    course2 = Course(name='Course Two')
    course3 = Course(name=Course Three)

    course1.members.append(Brady)
    course1.members.append(Paul)
    course2.members.append(Brady)
    course3.members.append(Paul)

    db.session.add(course1)
    db.session.add(course2)
    db.session.add(course3)
    db.session.commit()

<figure style="text-align: center;">
    <img src="README_img/Relationship.002.jpeg" alt="Many-to-Many Relationship" style="width: 80%; height: 80%"/>
    <figcaption style="display: block;">Many-to-Many Relationship</figcaption>
</figure>

<figure style="text-align: center;">
    <img src="README_img/Relationship.003.jpeg" alt="Many-to-Many Relationship Table" style="width: 80%; height: 80%"/>
    <figcaption style="display: block;">Many-to-Many Relationship Table</figcaption>
</figure>

## Code

#### Setting
------------------------------------------------------------------------

We leave the setting of the file at config.cfg.
In order to connect to the database, we specify the database path with prefix `mysql://` Put the database username and password `root:root` and followed by hostname, port and MySQL database name `localhost:3306/mydatabase`

config.cfg
```
SQLALCHEMY_DATABASE_URI = 'mysql://root:root@localhost:3306/mydatabase'
SQLALCHEMY_TRACK_MODIFICATIONS = False
DEBUG = True
```

Then in application.py, we can read the config by:

application.py
```
app.config.from_pyfile('config.cfg')
```

First we start an app initialized by Flask Framework `app = Flask(__name__)`

Then we can initialize the db by `db = SQLAlchemy(app)`

#### Schema Design
------------------------------------------------------------------------

We need to create members with some basic informations. In this case, we create member with id, username, password, email, and join_date. Each member can have orders and courses, however the different relationship between orders and courses is that the orders is One-to-many relationship and the courses is Many-to-Many relationship.

application.py
```
class Member(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(30), unique=True)
    password = db.Column(db.String(30))
    email = db.Column(db.String(50))
    join_date = db.Column(db.DateTime)

    orders = db.relationship('Order', backref='member', lazy='dynamic')
    courses = db.relationship('Course', secondary='user_courses', backref='members', lazy='dynamic')

    def __repr__(self):
        return '<Member %r>' % self.username
```

orders and courses colume are virtual colume, they are not existed in the Model. However, by using the backref, Order and Course can refer to the Member by **member** and **members**.

Order has id, price and member_id colume, member_id is a foreign_key which refer to the member instance.

application.py
```
class Order(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    price = db.Column(db.Integer)
    member_id = db.Column(db.Integer, db.ForeignKey('member.id'))
```

Course only has id and name cloumn, member_id is a foreign_key which refer to the member instance. By using the table, we can refer the course each member takes and vice versa.

application.py
```
class Course(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(20))

db.Table('user_courses',
    db.Column('member_id', db.Integer, db.ForeignKey('member.id')),
    db.Column('course_id', db.Integer, db.ForeignKey('course.id'))
    )
```

## Conclusion

In this project, we connect to local MySQL. Create members, orders and courses. Using One-to-Many relationship and Many-to-Many relationship to build the connections between them.

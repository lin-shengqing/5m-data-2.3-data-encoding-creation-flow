# Assignment

## Brief

Write the Python codes for the following questions.

## Instructions

Paste the answer as Python in the answer code section below each question.

### Question 1

Question: Implement a simple Thrift server and client that defines a `Student` struct with fields `name` (string), `age` (integer), and `courses` (list of strings). Include a service `School` with a method `enrollCourse` that takes a `Student` record and a course name, adds the course to the student's course list, and returns the updated `Student` record.

Answer:

```python
# Thrift schema (student.thrift)

%%writefile ../schema/student.thrift

struct Student {
  1: required string name,
  2: optional i64 age,
  3: optional list<string> courses
}

service School {
    Student enrollCourse(1: required Student student, 2: required list<string> courses)
}


# Thrift server (student_server.py)

%%writefile ../student_server.py
import thriftpy2
student_thrift = thriftpy2.load("./schema/student.thrift", module_name="student_thrift")

from thriftpy2.rpc import make_server

class School(object):
    def enrollCourse(self, student, courses):
        student.courses.extend(courses)
        return student

server = make_server(student_thrift.School, School(), client_timeout=None)
server.serve()


# Thrift client (student_client.py)

import thriftpy2
student_thrift = thriftpy2.load("../schema/student.thrift", module_name="student_thrift")

from thriftpy2.rpc import make_client

school = make_client(student_thrift.School, timeout=None)

martin = student_thrift.Student(
    name="Martin", age=20, courses=["DS", "AI"]
)

print(martin)

martin = school.enrollCourse(martin, ["Python", "Pandas"])

print(martin)
print(martin.courses)

```

### Question 2

Question: Implement a simple Protocol Buffers server and client that defines a `Book` message with fields `title` (string), `author` (string), and `page_count` (integer). Include a service `Library` with a method `checkoutBook` that takes a `Book` message and returns the same `Book` message.

Answer:

```python
# Protobuf schema (book.proto)

%%writefile ../schema/book.proto
syntax = "proto3";

message Book {
  string title = 1;
  string author = 2;
  optional int64 page_count = 3;
}

message BookRequest {
  Book book = 1;
  string author = 2;
  int64 page_count = 3;
}

service Library {
  rpc checkoutBook(BookRequest) returns (Book) {}
}


# Protobuf server (book_server.py)

%%writefile ../book_server.py
from concurrent import futures
import grpc
import book_pb2_grpc


class Library(book_pb2_grpc.LibraryServicer):

  def checkoutBook(self, request, context):
    request.book.author = request.author
    return request.book

server = grpc.server(futures.ThreadPoolExecutor(max_workers=2))
book_pb2_grpc.add_LibraryServicer_to_server(
    Library(), server)
server.add_insecure_port('[::]:50051')
server.start()
server.wait_for_termination()


# Protobuf client (book_client.py)

import sys
sys.path.append('..')
import grpc
import book_pb2
import book_pb2_grpc

def checkoutBook(stub, book, author):
    book = stub.checkoutBook(book_pb2.BookRequest(book=book, author=author))
    return book

with grpc.insecure_channel('localhost:50051') as channel:
    martin = book_pb2.Book(title='Martin Book', author='', page_count=67)
    author = "Martin"
    stub = book_pb2_grpc.LibraryStub(channel)
    martin = checkoutBook(stub, martin, author)
    print(martin)
    print(f"Book title: {martin.title}, Author: {martin.author}, Page Count: {martin.page_count}")
```

## Submission

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

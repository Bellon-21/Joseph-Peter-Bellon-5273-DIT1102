# Joseph-Peter-Bellon-5273-DIT1102
Object Oriented Programming Assignment

##Operations.py 
[operations.py](https://github.com/user-attachments/files/23417263/operations.py)
"""
operations.py
Core functions for the Mini Library Management System.
"""

from typing import Dict, List, Optional, Tuple

# ------------------ Data Structures ------------------
books: Dict[str, Dict] = {}
members: List[Dict] = []
genres: Tuple[str, ...] = ("Fiction", "Non-Fiction", "Sci-Fi", "Biography", "Mystery", "Romance")

MAX_BORROW = 3  # configurable borrow limit


# ------------------ Helper ------------------
def find_member(member_id: str) -> Optional[Dict]:
    """Find member by ID."""
    for m in members:
        if m["member_id"] == member_id:
            return m
    return None


# ------------------ Book Operations ------------------
def add_book(isbn: str, title: str, author: str, genre: str, total_copies: int) -> bool:
    """Add a book if ISBN unique and genre valid."""
    if isbn in books or genre not in genres or total_copies < 1:
        return False
    books[isbn] = {
        "title": title,
        "author": author,
        "genre": genre,
        "total_copies": total_copies,
        "available_copies": total_copies
    }
    return True


def search_books(query: str) -> List[Dict]:
    """Search by substring in title or author (case-insensitive)."""
    q = query.strip().lower()
    results = []
    for isbn, b in books.items():
        if q in b["title"].lower() or q in b["author"].lower():
            item = b.copy()
            item["isbn"] = isbn
            results.append(item)
    return results


def update_book(isbn: str, title: Optional[str] = None, author: Optional[str] = None,
                genre: Optional[str] = None, total_copies: Optional[int] = None) -> bool:
    """Update book details."""
    if isbn not in books:
        return False
    book = books[isbn]
    if genre is not None and genre not in genres:
        return False
    if title is not None:
        book["title"] = title
    if author is not None:
        book["author"] = author
    if genre is not None:
        book["genre"] = genre
    if total_copies is not None:
        if total_copies < 0:
            return False
        diff = total_copies - book["total_copies"]
        book["total_copies"] = total_copies
        book["available_copies"] = max(0, book["available_copies"] + diff)
    return True


def delete_book(isbn: str) -> bool:
    """Delete a book only if all copies are available."""
    if isbn not in books:
        return False
    book = books[isbn]
    if book["available_copies"] != book["total_copies"]:
        return False
    del books[isbn]
    return True


# ------------------ Member Operations ------------------
def add_member(member_id: str, name: str, email: str) -> bool:
    """Add a library member."""
    if find_member(member_id) is not None:
        return False
    members.append({
        "member_id": member_id,
        "name": name,
        "email": email,
        "borrowed_books": []
    })
    return True


def update_member(member_id: str, name: Optional[str] = None, email: Optional[str] = None) -> bool:
    """Update member details."""
    mem = find_member(member_id)
    if mem is None:
        return False
    if name is not None:
        mem["name"] = name
    if email is not None:
        mem["email"] = email
    return True


def delete_member(member_id: str) -> bool:
    """Delete a member only if they have returned all borrowed books."""
    mem = find_member(member_id)
    if mem is None or len(mem["borrowed_books"]) > 0:
        return False
    members.remove(mem)
    return True


# ------------------ Borrowing Operations ------------------
def borrow_book(member_id: str, isbn: str) -> bool:
    """Allow a member to borrow a book if available."""
    mem = find_member(member_id)
    if mem is None or isbn not in books:
        return False
    book = books[isbn]
    if book["available_copies"] <= 0 or len(mem["borrowed_books"]) >= MAX_BORROW:
        return False
    book["available_copies"] -= 1
    mem["borrowed_books"].append(isbn)
    return True


def return_book(member_id: str, isbn: str) -> bool:
    """Return a borrowed book."""
    mem = find_member(member_id)
    if mem is None or isbn not in mem["borrowed_books"]:
        return False
    mem["borrowed_books"].remove(isbn)
    if isbn in books:
        books[isbn]["available_copies"] = min(
            books[isbn]["total_copies"], books[isbn]["available_copies"] + 1)
    return True


# ------------------ Utility ------------------
def reset_library():
    """Clear all data (useful for tests/demo)."""
    books.clear()
    members.clear()
perations.py…]()


##demo.py
[demo.py](https://github.com/user-attachments/files/23417298/demo.py)

from operations import (
    genres, add_book, add_member, borrow_book, return_book, delete_book,
    delete_member, update_book, search_books, reset_library, books, members
)


def print_state():
    print("\n--- BOOKS ---")
    for isbn, b in books.items():
        print(isbn, b)
    print("\n--- MEMBERS ---")
    for m in members:
        print(m)


def main():
    reset_library()
    print("Genres available:", genres)

    # Add books
    print("Add book 978-001:", add_book("978-001", "Python Basics", "John Doe", "Fiction", 3))
    print("Add book 978-002:", add_book("978-002", "Data Science 101", "Jane Smith", "Non-Fiction", 2))
    print("Add book 978-003:", add_book("978-003", "Space Travel", "C. Astro", "Sci-Fi", 1))

    # Add members
    print("Add member M001:", add_member("M001", "Alice", "alice@example.com"))
    print("Add member M002:", add_member("M002", "Bob", "bob@example.com"))

    print_state()

    # Borrowing
    print("\nBorrow operations:")
    print("Alice borrows 978-001:", borrow_book("M001", "978-001"))
    print("Alice borrows 978-002:", borrow_book("M001", "978-002"))
    print("Alice borrows 978-003:", borrow_book("M001", "978-003"))
    print("Alice tries 4th borrow (should fail):", borrow_book("M001", "978-001"))
    print("Bob borrows 978-003 (should fail):", borrow_book("M002", "978-003"))

    print_state()

    # Return and re-borrow
    print("\nReturning and re-borrowing:")
    print("Alice returns 978-003:", return_book("M001", "978-003"))
    print("Bob borrows 978-003 now:", borrow_book("M002", "978-003"))

    # Update and delete attempts
    print("\nUpdates:")
    print("Update 978-002 title and copies:",
          update_book("978-002", title="Data Science Advanced", total_copies=3))
    print("Attempt delete 978-001 while borrowed (should fail):", delete_book("978-001"))

    # Member delete
    print("\nMember delete tests:")
    print("Delete Bob (should fail if borrowed):", delete_member("M002"))
    print("Bob returns 978-003:", return_book("M002", "978-003"))
    print("Delete Bob now (should succeed):", delete_member("M002"))

    print_state()

    # Search
    print("\nSearch for 'Data':", search_books("Data"))


if __name__ == "__main__":
    main()


##Test.py
[test.py](https://github.com/user-attachments/files/23417366/test.py)
"""
tests.py
Automated tests for the Mini Library Management System.
"""

from operations import (
    add_book, add_member, borrow_book, return_book,
    delete_book, delete_member, update_book, search_books, reset_library
)


def run_tests():
    reset_library()

    # Test 1 - add book
    assert add_book("100A", "Test Driven", "T Tester", "Fiction", 2) is True
    # Test 2 - duplicate ISBN
    assert add_book("100A", "Duplicate", "Someone", "Fiction", 1) is False
    # Test 3 - add member and borrow
    assert add_member("U1", "User One", "u1@mail.com") is True
    assert borrow_book("U1", "100A") is True
    # Test 4 - cannot delete a book when borrowed
    assert delete_book("100A") is False
    # Test 5 - return then delete
    assert return_book("U1", "100A") is True
    assert delete_book("100A") is True
    # Test 6 - delete member with borrowed books
    add_book("200B", "Another", "Author", "Non-Fiction", 1)
    add_member("U2", "User Two", "u2@mail.com")
    assert borrow_book("U2", "200B") is True
    assert delete_member("U2") is False
    # Test 7 - update book
    assert update_book("200B", title="Another 2", total_copies=2) is True
    sb = search_books("Another")
    assert isinstance(sb, list) and sb and sb[0]["title"] == "Another 2"

    print(" All tests passed successfully!")


if __name__ == "__main__":
    run_tests()


##UML Diagram
![WhatsApp Image 2025-11-05 at 13 10 58_b24d8c27](https://github.com/user-attachments/assets/11fd3356-5395-4e71-8b07-43fce7451888)

##Rationale
[Design Rationale for the Mini Library Management System.docx](https://github.com/user-attachments/files/23360750/Design.Rationale.for.the.Mini.Library.Management.System.docx)

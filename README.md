# TASK-3-
import sqlite3
import os

class FlashcardApp:
    def __init__(self, db_name="flashcards.db"):
        self.db_name = db_name
        self.conn = sqlite3.connect(self.db_name)
        self.cursor = self.conn.cursor()
        self.create_tables()

    def create_tables(self):
        self.cursor.execute("""
            CREATE TABLE IF NOT EXISTS flashcards (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                question TEXT NOT NULL,
                answer TEXT NOT NULL
            )
        """)
        self.cursor.execute("""
            CREATE TABLE IF NOT EXISTS quiz_results (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                score INTEGER NOT NULL,
                total_questions INTEGER NOT NULL
            )
        """)
        self.conn.commit()

    def add_flashcard(self, question, answer):
        self.cursor.execute("INSERT INTO flashcards (question, answer) VALUES (?, ?)", (question, answer))
        self.conn.commit()
        print("Flashcard added successfully!")

    def list_flashcards(self):
        self.cursor.execute("SELECT * FROM flashcards")
        flashcards = self.cursor.fetchall()
        if not flashcards:
            print("No flashcards found.")
            return

        for card in flashcards:
            print(f"ID: {card[0]}, Question: {card[1]}, Answer: {card[2]}")

    def take_quiz(self):
        self.cursor.execute("SELECT * FROM flashcards ORDER BY RANDOM()")
        flashcards = self.cursor.fetchall()
        if not flashcards:
            print("No flashcards available for a quiz.")
            return

        score = 0
        total_questions = len(flashcards)

        for card in flashcards:
            print(f"\nQuestion: {card[1]}")
            user_answer = input("Your answer: ")
            if user_answer.lower() == card[2].lower():
                print("Correct!")
                score += 1
            else:
                print(f"Incorrect. The correct answer is: {card[2]}")

        print(f"\nQuiz complete! Your score: {score}/{total_questions}")
        self.save_quiz_result(score, total_questions)

    def save_quiz_result(self, score, total_questions):
        self.cursor.execute("INSERT INTO quiz_results (score, total_questions) VALUES (?, ?)", (score, total_questions))
        self.conn.commit()
        print("Quiz result saved.")

    def view_quiz_results(self):
        self.cursor.execute("SELECT * FROM quiz_results")
        results = self.cursor.fetchall()
        if not results:
            print("No quiz results found.")
            return

        for result in results:
            print(f"ID: {result[0]}, Score: {result[1]}/{result[2]}")

    def close(self):
        self.conn.close()

def main():
    app = FlashcardApp()

    while True:
        print("\nFlashcard App Menu:")
        print("1. Add Flashcard")
        print("2. List Flashcards")
        print("3. Take Quiz")
        print("4. View Quiz Results")
        print("5. Exit")

        choice = input("Enter your choice: ")

        if choice == "1":
            question = input("Enter question: ")
            answer = input("Enter answer: ")
            app.add_flashcard(question, answer)
        elif choice == "2":
            app.list_flashcards()
        elif choice == "3":
            app.take_quiz()
        elif choice == "4":
            app.view_quiz_results()
        elif choice == "5":
            app.close()
            print("Exiting...")
            break
        else:
            print("Invalid choice. Please try again.")

if __name__ == "__main__":
    main()


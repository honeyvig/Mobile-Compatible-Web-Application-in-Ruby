# Mobile-Compatible-Web-Application-in-Ruby
Build a web platform where instructors can create accounts, generate/edit questions, and offer quizzes to students via a unique URL. Students will have their own accounts where they can log in, take quizzes before and after class (pre- and post-class tests), and view their results for each test. The platform should allow instructors to easily manage question sets, and students should be able to access their quizzes and track their performance over time.

Core Features:
Instructor Portal:
Account Management:
Instructors should be able to sign up, log in, and reset passwords.
A dashboard where instructors can manage their quizzes, question banks, and classes.
Question Creation and Editing:



Instructors should be able to create quizzes with various question types (multiple-choice, true/false, short answer).
Editing Functionality: Instructors can edit questions and answers even after creating a quiz, as long as no students have started the quiz.
Pre- and Post-Class Test:



Instructors should be able to mark certain quizzes as pre- and post-class tests, where the same test is taken twice:
Pre-Class Test: Taken by students before the class.
Post-Class Test: Taken by students after the class.
Both tests will use the same questions and be accessible via the same unique URL.
Instructors can specify when the pre-class test should be available and when it should switch to the post-class test.
Quiz Organization and Sharing:



Instructors can organize quizzes by class or assessment.
For pre- and post-class tests, the system will automatically generate a single URL for both versions of the quiz.
Instructors can assign quizzes to specific classes or individual students.
Results and Analytics for Instructors:



Instructors should be able to view and compare student performance on the pre- and post-class tests.
The results dashboard should show individual student scores, as well as class-wide performance, including:
Pre-class test scores.
Post-class test scores.
Improvement or change between the two.
Instructors should be able to download these results for further analysis.
Student Portal:
Student Account Management:



Students should be able to sign up, log in, and reset passwords.
Students will have a dashboard where they can:
View upcoming quizzes and class assessments.
See completed quizzes, including pre- and post-class tests.
Pre- and Post-Class Test Experience:



Students will take the pre-class test before the class begins, and their results will be recorded.
After the class, students will take the post-class test through the same link, and their results will again be recorded.
Students should be able to view both their pre-class and post-class results in the dashboard, side-by-side, to see how they’ve improved or what concepts they need to focus on.
Gradebook:



The gradebook will display:
The name of each quiz or assessment.
Pre-class and post-class test results (for relevant quizzes).
Improvement metrics (e.g., increase in score percentage).
Instructor feedback.
Students will be able to review past quizzes and results at any time.
Notifications:



Students will receive notifications when new quizzes are available or when their post-class test results are ready to view.
Additional Features:
Timed Quizzes: Instructors can create timed quizzes, including pre- and post-class tests.
Question Bank: Instructors can save questions into a reusable bank to generate future quizzes.
Security: Proper security features, including JWT-based authentication, HTTPS, and encryption for sensitive data.
Result Release: Instructors can decide when to release quiz results to students (immediately after submission or later).
============================
Below is a Ruby on Rails code outline for building the described web platform. This is a high-level implementation that can be extended based on your needs.
1. Rails Models

Create models to represent the core entities: User (for instructors and students), Quiz, Question, Result, and Class.

# app/models/user.rb
class User < ApplicationRecord
  has_secure_password
  has_many :quizzes, foreign_key: :instructor_id, dependent: :destroy
  has_many :results, dependent: :destroy

  enum role: { instructor: 0, student: 1 }

  validates :email, presence: true, uniqueness: true
end

# app/models/quiz.rb
class Quiz < ApplicationRecord
  belongs_to :instructor, class_name: "User"
  has_many :questions, dependent: :destroy
  has_many :results, dependent: :destroy

  validates :title, :pre_class_start_time, :post_class_start_time, presence: true
end

# app/models/question.rb
class Question < ApplicationRecord
  belongs_to :quiz

  validates :content, presence: true
  enum question_type: { multiple_choice: 0, true_false: 1, short_answer: 2 }
end

# app/models/result.rb
class Result < ApplicationRecord
  belongs_to :user
  belongs_to :quiz

  validates :score, presence: true
end

2. User Authentication

Use Devise or has_secure_password for authentication. Below is an example with has_secure_password:

rails generate migration add_password_digest_to_users password_digest:string

Update User model with has_secure_password.
3. Controllers

Create controllers for managing users, quizzes, questions, and results.
Users Controller

# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def signup
    @user = User.new(user_params)
    if @user.save
      render json: { message: "Signup successful" }
    else
      render json: { errors: @user.errors.full_messages }, status: :unprocessable_entity
    end
  end

  private

  def user_params
    params.require(:user).permit(:email, :password, :role)
  end
end

Quizzes Controller

# app/controllers/quizzes_controller.rb
class QuizzesController < ApplicationController
  def create
    @quiz = current_user.quizzes.build(quiz_params)
    if @quiz.save
      render json: @quiz
    else
      render json: { errors: @quiz.errors.full_messages }, status: :unprocessable_entity
    end
  end

  def index
    @quizzes = current_user.quizzes
    render json: @quizzes
  end

  private

  def quiz_params
    params.require(:quiz).permit(:title, :pre_class_start_time, :post_class_start_time)
  end
end

Questions Controller

# app/controllers/questions_controller.rb
class QuestionsController < ApplicationController
  def create
    @quiz = Quiz.find(params[:quiz_id])
    @question = @quiz.questions.build(question_params)
    if @question.save
      render json: @question
    else
      render json: { errors: @question.errors.full_messages }, status: :unprocessable_entity
    end
  end

  private

  def question_params
    params.require(:question).permit(:content, :question_type, :correct_answer, :options)
  end
end

4. Routes

Define routes for the application.

# config/routes.rb
Rails.application.routes.draw do
  resources :users, only: [:create]
  resources :quizzes do
    resources :questions, only: [:create]
  end
  resources :results, only: [:create, :index]
end

5. Frontend Integration

Integrate a frontend framework like React or Vue.js to build user-friendly interfaces for both instructors and students. Use REST APIs to interact with the backend.
6. Pre- and Post-Class Tests

Logic for pre- and post-class test toggling can be implemented by comparing pre_class_start_time and post_class_start_time with the current time.

def quiz_access
  if Time.now < @quiz.pre_class_start_time
    render json: { message: "Pre-class test not yet available" }
  elsif Time.now.between?(@quiz.pre_class_start_time, @quiz.post_class_start_time)
    render json: { message: "Pre-class test available" }
  else
    render json: { message: "Post-class test available" }
  end
end

7. Analytics and Results

Generate analytics using ActiveRecord queries and present them on the instructor dashboard.
8. Security

Ensure user data is secure:

    Use HTTPS.
    Implement role-based access control.
    Validate and sanitize input parameters.

This setup provides a solid foundation for your platform. Depending on the scope, you can expand functionalities like notifications, gradebook, and advanced analytics.

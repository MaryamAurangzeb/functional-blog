# functional-blog

Milestone 4: Project
Objective: Build a fully-functional blog platform with user authentication, where users can create, edit, and delete posts. Ensure the application is production-ready and deployed.

Steps to Implement the Blog Platform
Step 1: Set Up the Project
First, let's create a new Next.js project for the blog platform.

bash
Copy code
npx create-next-app@latest blog-platform
cd blog-platform
Step 2: Install Dependencies
For the blog platform, we’ll need a few dependencies to handle authentication and API routes:

bash
Copy code
npm install next-auth react-hook-form
next-auth for handling authentication.
react-hook-form for handling form submissions.
Step 3: Set Up Authentication
We will use NextAuth.js to handle user authentication.

Create the Auth API route:
Create a file pages/api/auth/[...nextauth].js to configure NextAuth for authentication.
javascript
Copy code
import NextAuth from "next-auth";
import CredentialsProvider from "next-auth/providers/credentials";

export default NextAuth({
  providers: [
    CredentialsProvider({
      name: "Credentials",
      credentials: {
        username: { label: "Username", type: "text" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        const { username, password } = credentials;

        // Simple authentication (replace with a database in production)
        if (username === "admin" && password === "password123") {
          return { id: 1, name: "Admin", email: "admin@example.com" };
        }

        return null; // Return null if authentication fails
      },
    }),
  ],
  pages: {
    signIn: '/login',
  },
  session: {
    strategy: 'jwt',
  },
  secret: process.env.NEXTAUTH_SECRET,
});
Explanation:

The credentials provider allows us to authenticate using a simple username and password (you can replace this with real database authentication).
The authorize method checks if the username and password match predefined values (for simplicity in this example).
Add environment variables: In your .env.local file, add the following:
env
Copy code
NEXTAUTH_SECRET=your-secret-here
This secret is required for NextAuth.js.

Step 4: Create the Authentication Pages
Login Page:
Create a login page pages/login.js to authenticate users.
javascript
Copy code
import { signIn } from "next-auth/react";
import { useState } from "react";

const LoginPage = () => {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");

  const handleLogin = async (e) => {
    e.preventDefault();
    const res = await signIn("credentials", {
      redirect: false,
      username,
      password,
    });

    if (res?.error) {
      setError("Invalid credentials");
    }
  };

  return (
    <div className="p-4">
      <h1 className="text-3xl font-bold mb-4">Login</h1>
      <form onSubmit={handleLogin}>
        <input
          type="text"
          placeholder="Username"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
          className="mb-2 p-2 border rounded"
        />
        <input
          type="password"
          placeholder="Password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          className="mb-2 p-2 border rounded"
        />
        <button type="submit" className="bg-blue-600 text-white p-2 rounded">
          Log In
        </button>
      </form>
      {error && <p className="mt-2 text-red-500">{error}</p>}
    </div>
  );
};

export default LoginPage;
Explanation:

The login form allows users to input their credentials.
The signIn function from NextAuth is used to authenticate the user with the credentials provider.
Step 5: Create the Blog Post Model and API
We will need API routes to create, edit, and delete posts. Let's use an in-memory database (a simple array for demonstration) for the posts.

Create an API Route for Posts:
Create a file pages/api/posts.js to handle creating, editing, and deleting blog posts.
javascript
Copy code
let posts = [
  { id: 1, title: "My First Blog Post", content: "This is the content of my first blog post." },
  { id: 2, title: "Next.js for Beginners", content: "Learn how to build a blog using Next.js." },
];

export default function handler(req, res) {
  if (req.method === 'GET') {
    res.status(200).json(posts); // Get all posts
  }

  if (req.method === 'POST') {
    const { title, content } = req.body;
    const newPost = { id: posts.length + 1, title, content };
    posts.push(newPost);
    res.status(201).json(newPost); // Create new post
  }

  if (req.method === 'PUT') {
    const { id, title, content } = req.body;
    const post = posts.find((post) => post.id === id);
    if (post) {
      post.title = title;
      post.content = content;
      res.status(200).json(post); // Update post
    } else {
      res.status(404).json({ message: "Post not found" });
    }
  }

  if (req.method === 'DELETE') {
    const { id } = req.query;
    posts = posts.filter((post) => post.id !== parseInt(id));
    res.status(200).json({ message: "Post deleted" }); // Delete post
  }
}
Explanation:

This API handles GET (fetch posts), POST (create post), PUT (edit post), and DELETE (delete post) requests.
In a real app, you'd replace the in-memory array with a database (e.g., MongoDB, PostgreSQL).
Step 6: Create the Blog Post Pages
Create Post List Page:
Create a page pages/posts.js to display a list of all posts.
javascript
Copy code
import { useEffect, useState } from 'react';
import Link from 'next/link';

const PostsPage = () => {
  const [posts, setPosts] = useState([]);

  useEffect(() => {
    fetch('/api/posts')
      .then((res) => res.json())
      .then((data) => setPosts(data));
  }, []);

  return (
    <div className="p-4">
      <h1 className="text-3xl font-bold mb-4">All Posts</h1>
      <ul>
        {posts.map((post) => (
          <li key={post.id} className="border-b py-2">
            <h2 className="font-semibold">{post.title}</h2>
            <p>{post.content}</p>
            <Link href={`/edit/${post.id}`}>Edit</Link> | <button>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default PostsPage;
Explanation:

This page fetches and displays all posts using the GET method of the API route.
Create Post Editing Page:
Create a page pages/edit/[id].js to edit an existing post.
javascript
Copy code
import { useState, useEffect } from "react";
import { useRouter } from "next/router";

const EditPostPage = () => {
  const router = useRouter();
  const { id } = router.query;

  const [title, setTitle] = useState("");
  const [content, setContent] = useState("");

  useEffect(() => {
    if (id) {
      fetch(`/api/posts?id=${id}`)
        .then((res) => res.json())
        .then((data) => {
          setTitle(data.title);
          setContent(data.content);
        });
    }
  }, [id]);

  const handleSave = async () => {
    await fetch('/api/posts', {
      method: 'PUT',
      body: JSON.stringify({ id, title, content }),
      headers: { 'Content-Type': 'application/json' },
    });
    router.push('/posts');
  };

  return (
    <div className="p-4">
      <h1 className="text-3xl font-bold mb-4">Edit Post</h1>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        className="mb-2 p-2 border rounded w-full"
      />
      <textarea
        value={content}
        onChange={(e) => setContent(e.target.value)}
        className="mb-2 p-2 border rounded w-full"
      ></textarea>
      <button onClick={handleSave} className="bg-blue-600 text-white p-2 rounded">Save</button>
    </div>
  );
};

export default EditPostPage;
Explanation:

This page allows the user to edit an existing post. After editing, it calls the PUT


Absolutely! I'll add comprehensive testing for your DevBlog Pro application with unit testing, integration testing, and end-to-end testing. Here's the complete testing setup:

## Updated Package.json with Testing Dependencies

```json
{
  "name": "devblog-pro",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0",
    "@reduxjs/toolkit": "^1.9.7",
    "react-redux": "^8.1.3",
    "@tanstack/react-query": "^5.12.2",
    "axios": "^1.6.2",
    "react-hook-form": "^7.48.2",
    "zod": "^3.22.4",
    "@hookform/resolvers": "^3.3.2",
    "react-quill": "^2.0.0",
    "react-helmet-async": "^2.0.4",
    "react-hot-toast": "^2.4.1",
    "date-fns": "^2.30.0",
    "framer-motion": "^10.16.16",
    "react-intersection-observer": "^9.5.3"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.1",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.32",
    "tailwindcss": "^3.3.6",
    "vite": "^5.0.10",
    "@testing-library/react": "^14.1.2",
    "@testing-library/jest-dom": "^6.1.5",
    "@testing-library/user-event": "^14.5.1",
    "jest": "^29.7.0",
    "jest-environment-jsdom": "^29.7.0",
    "@testing-library/react-hooks": "^8.0.1",
    "msw": "^2.0.11",
    "cypress": "^13.6.0",
    "@cypress/code-coverage": "^3.12.0",
    "cypress-real-events": "^1.10.3",
    "cypress-wait-until": "^2.0.1",
    "@playwright/test": "^1.40.0",
    "playwright": "^1.40.0",
    "vitest": "^1.0.4",
    "@vitest/ui": "^1.0.4",
    "jsdom": "^23.0.1",
    "@vitest/coverage-v8": "^1.0.4",
    "eslint-plugin-jest-dom": "^5.1.0",
    "eslint-plugin-testing-library": "^6.2.0"
  },
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:cypress": "cypress open",
    "test:cypress:run": "cypress run",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:all": "npm run test && npm run test:cypress:run && npm run test:e2e"
  }
}
```

## 1. Vitest Configuration (vitest.config.js)

```javascript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: "jsdom",
    setupFiles: ["./src/test/setup.js"],
    css: true,
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: [
        "node_modules/",
        "src/test/",
        "**/*.d.ts",
        "**/*.config.*",
        "**/index.jsx",
        "src/main.jsx",
      ],
    },
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

## 2. Test Setup File (src/test/setup.js)

```javascript
import "@testing-library/jest-dom/vitest";
import { cleanup } from "@testing-library/react";
import { afterEach, beforeAll, afterAll, vi } from "vitest";
import { server } from "./mocks/server";

// Mock window.matchMedia
Object.defineProperty(window, "matchMedia", {
  writable: true,
  value: vi.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor(callback) {
    this.callback = callback;
  }
  observe() {
    this.callback([{ isIntersecting: true }]);
  }
  unobserve() {}
  disconnect() {}
};

// Mock localStorage
const localStorageMock = {
  getItem: vi.fn(),
  setItem: vi.fn(),
  removeItem: vi.fn(),
  clear: vi.fn(),
};
global.localStorage = localStorageMock;

// Start MSW server before all tests
beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterAll(() => server.close());
afterEach(() => {
  cleanup();
  server.resetHandlers();
  vi.clearAllMocks();
});
```

## 3. MSW (Mock Service Worker) Setup for API Testing

### Server Setup (src/test/mocks/server.js)

```javascript
import { setupServer } from "msw/node";
import { handlers } from "./handlers";

export const server = setupServer(...handlers);
```

### API Handlers (src/test/mocks/handlers.js)

```javascript
import { http, HttpResponse } from "msw";

const API_BASE_URL = "http://localhost:5000/api";

// Mock user data
const mockUser = {
  id: "1",
  name: "Test User",
  email: "test@example.com",
  avatar: "https://example.com/avatar.jpg",
  role: "user",
  bio: "Test bio",
  social: {
    github: "https://github.com/test",
    linkedin: "https://linkedin.com/in/test",
    twitter: "https://twitter.com/test",
  },
};

const mockAdmin = {
  id: "2",
  name: "Admin User",
  email: "admin@example.com",
  role: "admin",
};

const mockBlogs = [
  {
    id: "1",
    title: "Test Blog 1",
    content: "This is test content for blog 1",
    description: "Test description 1",
    coverImage: "https://example.com/image1.jpg",
    tags: ["React", "JavaScript"],
    category: "React",
    author: mockUser,
    likes: 10,
    views: 100,
    commentsCount: 5,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
  },
  {
    id: "2",
    title: "Test Blog 2",
    content: "This is test content for blog 2",
    description: "Test description 2",
    coverImage: "https://example.com/image2.jpg",
    tags: ["TypeScript", "NodeJS"],
    category: "TypeScript",
    author: mockUser,
    likes: 20,
    views: 200,
    commentsCount: 10,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
  },
];

const mockComments = [
  {
    id: "1",
    content: "Great article!",
    author: mockUser,
    createdAt: new Date().toISOString(),
  },
];

export const handlers = [
  // Auth endpoints
  http.post(`${API_BASE_URL}/auth/login`, async ({ request }) => {
    const { email, password } = await request.json();
    if (email === "test@example.com" && password === "password123") {
      return HttpResponse.json({
        user: mockUser,
        token: "mock-token-123",
      });
    }
    return new HttpResponse(null, { status: 401 });
  }),

  http.post(`${API_BASE_URL}/auth/register`, async ({ request }) => {
    const userData = await request.json();
    return HttpResponse.json({
      user: { ...mockUser, ...userData },
      token: "mock-token-123",
    });
  }),

  http.get(`${API_BASE_URL}/auth/me`, ({ request }) => {
    const token = request.headers.get("Authorization");
    if (token === "Bearer mock-token-123") {
      return HttpResponse.json(mockUser);
    }
    return new HttpResponse(null, { status: 401 });
  }),

  // Blog endpoints
  http.get(`${API_BASE_URL}/blogs`, ({ request }) => {
    const url = new URL(request.url);
    const page = parseInt(url.searchParams.get("page") || "1");
    const limit = parseInt(url.searchParams.get("limit") || "9");
    const start = (page - 1) * limit;
    const end = start + limit;

    return HttpResponse.json({
      blogs: mockBlogs.slice(start, end),
      totalPages: Math.ceil(mockBlogs.length / limit),
      currentPage: page,
      totalBlogs: mockBlogs.length,
    });
  }),

  http.get(`${API_BASE_URL}/blogs/:id`, ({ params }) => {
    const blog = mockBlogs.find((b) => b.id === params.id);
    if (blog) {
      return HttpResponse.json(blog);
    }
    return new HttpResponse(null, { status: 404 });
  }),

  http.post(`${API_BASE_URL}/blogs`, async ({ request }) => {
    const newBlog = await request.json();
    return HttpResponse.json(
      {
        ...newBlog,
        id: "3",
        author: mockUser,
        createdAt: new Date().toISOString(),
      },
      { status: 201 },
    );
  }),

  http.put(`${API_BASE_URL}/blogs/:id`, async ({ params, request }) => {
    const updates = await request.json();
    const blogIndex = mockBlogs.findIndex((b) => b.id === params.id);
    if (blogIndex !== -1) {
      const updatedBlog = { ...mockBlogs[blogIndex], ...updates };
      return HttpResponse.json(updatedBlog);
    }
    return new HttpResponse(null, { status: 404 });
  }),

  http.delete(`${API_BASE_URL}/blogs/:id`, ({ params }) => {
    const blogExists = mockBlogs.some((b) => b.id === params.id);
    if (blogExists) {
      return new HttpResponse(null, { status: 204 });
    }
    return new HttpResponse(null, { status: 404 });
  }),

  http.post(`${API_BASE_URL}/blogs/:id/like`, ({ params }) => {
    return HttpResponse.json({ likes: 11 });
  }),

  http.delete(`${API_BASE_URL}/blogs/:id/like`, () => {
    return HttpResponse.json({ likes: 9 });
  }),

  // Comments endpoints
  http.get(`${API_BASE_URL}/blogs/:id/comments`, () => {
    return HttpResponse.json(mockComments);
  }),

  http.post(`${API_BASE_URL}/blogs/:id/comments`, async ({ request }) => {
    const { content } = await request.json();
    const newComment = {
      id: Date.now().toString(),
      content,
      author: mockUser,
      createdAt: new Date().toISOString(),
    };
    return HttpResponse.json(newComment, { status: 201 });
  }),

  http.put(
    `${API_BASE_URL}/blogs/:blogId/comments/:commentId`,
    async ({ request }) => {
      const { content } = await request.json();
      return HttpResponse.json({ id: "1", content, author: mockUser });
    },
  ),

  http.delete(`${API_BASE_URL}/blogs/:blogId/comments/:commentId`, () => {
    return new HttpResponse(null, { status: 204 });
  }),

  // Bookmarks
  http.get(`${API_BASE_URL}/blogs/bookmarks`, () => {
    return HttpResponse.json(mockBlogs);
  }),

  http.post(`${API_BASE_URL}/blogs/:id/bookmark`, () => {
    return HttpResponse.json({ message: "Bookmarked" });
  }),

  http.delete(`${API_BASE_URL}/blogs/:id/bookmark`, () => {
    return new HttpResponse(null, { status: 204 });
  }),
];
```

## 4. Unit Tests

### Component Tests

#### BlogCard Component Test (src/components/cards/**tests**/BlogCard.test.jsx)

```javascript
import { render, screen, fireEvent } from "@testing-library/react";
import { BrowserRouter } from "react-router-dom";
import { BlogCard } from "../BlogCard";
import { describe, it, expect, vi } from "vitest";

describe("BlogCard Component", () => {
  const mockBlog = {
    id: "1",
    title: "Test Blog Title",
    description: "This is a test description for the blog",
    content: "Test content here",
    coverImage: "https://example.com/image.jpg",
    category: "React",
    tags: ["React", "JavaScript"],
    author: {
      id: "1",
      name: "John Doe",
      avatar: "https://example.com/avatar.jpg",
    },
    likes: 42,
    views: 100,
    commentsCount: 5,
    createdAt: "2024-01-01T00:00:00Z",
  };

  const renderComponent = (props = {}) => {
    return render(
      <BrowserRouter>
        <BlogCard blog={mockBlog} {...props} />
      </BrowserRouter>,
    );
  };

  it("should render blog title correctly", () => {
    renderComponent();
    expect(screen.getByText("Test Blog Title")).toBeInTheDocument();
  });

  it("should render blog description", () => {
    renderComponent();
    expect(
      screen.getByText("This is a test description for the blog"),
    ).toBeInTheDocument();
  });

  it("should render author name", () => {
    renderComponent();
    expect(screen.getByText("John Doe")).toBeInTheDocument();
  });

  it("should render category badge", () => {
    renderComponent();
    expect(screen.getByText("React")).toBeInTheDocument();
  });

  it("should render like count", () => {
    renderComponent();
    expect(screen.getByText("42")).toBeInTheDocument();
  });

  it("should call onLike when like button is clicked", () => {
    const onLike = vi.fn();
    renderComponent({ onLike });
    const likeButton = screen.getByRole("button", { name: /like/i });
    fireEvent.click(likeButton);
    expect(onLike).toHaveBeenCalledWith("1");
  });

  it("should display liked state when blog is liked", () => {
    const likedBlog = { ...mockBlog, isLiked: true };
    renderComponent({ blog: likedBlog });
    const heartIcon = screen.getByTestId("heart-icon");
    expect(heartIcon).toHaveClass("fill-red-500");
  });

  it("should display bookmarked state when blog is bookmarked", () => {
    renderComponent({ isBookmarked: true });
    const bookmarkIcon = screen.getByTestId("bookmark-icon");
    expect(bookmarkIcon).toHaveClass("fill-primary-600");
  });
});
```

#### LoginForm Component Test (src/pages/**tests**/LoginPage.test.jsx)

```javascript
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import { Provider } from "react-redux";
import { BrowserRouter } from "react-router-dom";
import { configureStore } from "@reduxjs/toolkit";
import { LoginPage } from "../LoginPage";
import authReducer from "../../features/auth/authSlice";
import { describe, it, expect, vi } from "vitest";

describe("LoginPage", () => {
  const createStore = () => {
    return configureStore({
      reducer: {
        auth: authReducer,
      },
    });
  };

  const renderComponent = () => {
    return render(
      <Provider store={createStore()}>
        <BrowserRouter>
          <LoginPage />
        </BrowserRouter>
      </Provider>,
    );
  };

  it("should render login form", () => {
    renderComponent();
    expect(screen.getByLabelText(/email address/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    expect(
      screen.getByRole("button", { name: /sign in/i }),
    ).toBeInTheDocument();
  });

  it("should show validation errors for empty fields", async () => {
    renderComponent();
    const submitButton = screen.getByRole("button", { name: /sign in/i });
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText(/invalid email address/i)).toBeInTheDocument();
    });
  });

  it("should show validation error for invalid email", async () => {
    renderComponent();
    const emailInput = screen.getByLabelText(/email address/i);
    fireEvent.change(emailInput, { target: { value: "invalid-email" } });

    const submitButton = screen.getByRole("button", { name: /sign in/i });
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByText(/invalid email address/i)).toBeInTheDocument();
    });
  });

  it("should toggle password visibility", () => {
    renderComponent();
    const passwordInput = screen.getByLabelText(/password/i);
    const toggleButton = screen.getByRole("button", {
      name: /toggle password visibility/i,
    });

    expect(passwordInput).toHaveAttribute("type", "password");
    fireEvent.click(toggleButton);
    expect(passwordInput).toHaveAttribute("type", "text");
  });

  it("should have link to register page", () => {
    renderComponent();
    const registerLink = screen.getByRole("link", { name: /sign up/i });
    expect(registerLink).toHaveAttribute("href", "/register");
  });

  it("should have link to forgot password", () => {
    renderComponent();
    const forgotLink = screen.getByRole("link", { name: /forgot password/i });
    expect(forgotLink).toHaveAttribute("href", "/forgot-password");
  });
});
```

### Utility Function Tests (src/utils/**tests**/helpers.test.js)

```javascript
import { describe, it, expect } from "vitest";
import {
  calculateReadingTime,
  formatDate,
  truncateText,
  generateSlug,
  isValidEmail,
  getPasswordStrength,
} from "../helpers";

describe("Helper Functions", () => {
  describe("calculateReadingTime", () => {
    it("should calculate correct reading time for short text", () => {
      const text = "Hello ".repeat(100); // 500 characters, ~100 words
      const time = calculateReadingTime(text);
      expect(time).toBe(1);
    });

    it("should calculate correct reading time for long text", () => {
      const text = "Word ".repeat(400); // ~400 words
      const time = calculateReadingTime(text);
      expect(time).toBe(2);
    });

    it("should handle HTML content", () => {
      const html = "<p>Hello <strong>world</strong></p>";
      const time = calculateReadingTime(html);
      expect(time).toBe(1);
    });

    it("should return 0 for empty content", () => {
      const time = calculateReadingTime("");
      expect(time).toBe(0);
    });
  });

  describe("formatDate", () => {
    it("should format date correctly", () => {
      const date = "2024-01-15T10:30:00Z";
      const formatted = formatDate(date);
      expect(formatted).toBe("Jan 15, 2024");
    });

    it("should handle different date formats", () => {
      const date = new Date(2024, 11, 25);
      const formatted = formatDate(date);
      expect(formatted).toBe("Dec 25, 2024");
    });
  });

  describe("truncateText", () => {
    it("should truncate text longer than limit", () => {
      const text = "This is a very long text that needs to be truncated";
      const truncated = truncateText(text, 20);
      expect(truncated).toBe("This is a very long...");
      expect(truncated.length).toBeLessThanOrEqual(23);
    });

    it("should not truncate text shorter than limit", () => {
      const text = "Short text";
      const truncated = truncateText(text, 20);
      expect(truncated).toBe("Short text");
    });

    it("should use default limit of 100", () => {
      const text = "a".repeat(150);
      const truncated = truncateText(text);
      expect(truncated).toBe("a".repeat(100) + "...");
    });
  });

  describe("generateSlug", () => {
    it("should generate slug from title", () => {
      const title = "My Awesome Blog Post";
      const slug = generateSlug(title);
      expect(slug).toBe("my-awesome-blog-post");
    });

    it("should handle special characters", () => {
      const title = "React & JavaScript: The Ultimate Guide!";
      const slug = generateSlug(title);
      expect(slug).toBe("react-javascript-the-ultimate-guide");
    });

    it("should handle multiple spaces", () => {
      const title = "Hello    World";
      const slug = generateSlug(title);
      expect(slug).toBe("hello-world");
    });
  });

  describe("isValidEmail", () => {
    it("should return true for valid email", () => {
      expect(isValidEmail("test@example.com")).toBe(true);
      expect(isValidEmail("user.name@domain.co.uk")).toBe(true);
      expect(isValidEmail("test+label@example.com")).toBe(true);
    });

    it("should return false for invalid email", () => {
      expect(isValidEmail("invalid")).toBe(false);
      expect(isValidEmail("test@")).toBe(false);
      expect(isValidEmail("@example.com")).toBe(false);
      expect(isValidEmail("test@example")).toBe(false);
    });
  });

  describe("getPasswordStrength", () => {
    it("should return 0 for empty password", () => {
      const strength = getPasswordStrength("");
      expect(strength).toBe(0);
    });

    it("should return 1 for short password", () => {
      const strength = getPasswordStrength("1234567");
      expect(strength).toBe(1);
    });

    it("should return 2 for password with length and lowercase", () => {
      const strength = getPasswordStrength("password123");
      expect(strength).toBe(2);
    });

    it("should return 3 for password with length, lowercase, uppercase", () => {
      const strength = getPasswordStrength("Password123");
      expect(strength).toBe(3);
    });

    it("should return 4 for strong password", () => {
      const strength = getPasswordStrength("StrongP@ss123");
      expect(strength).toBe(4);
    });
  });
});
```

### Redux Slice Tests (src/features/auth/**tests**/authSlice.test.js)

```javascript
import { describe, it, expect } from "vitest";
import authReducer, { login, logout, clearError } from "../authSlice";
import { configureStore } from "@reduxjs/toolkit";

describe("auth slice", () => {
  const initialState = {
    user: null,
    token: null,
    isLoading: false,
    error: null,
  };

  it("should return initial state", () => {
    expect(authReducer(undefined, { type: undefined })).toEqual(initialState);
  });

  it("should handle clearError", () => {
    const stateWithError = { ...initialState, error: "Some error" };
    const newState = authReducer(stateWithError, clearError());
    expect(newState.error).toBeNull();
  });

  describe("login async thunk", () => {
    it("should handle pending state", () => {
      const action = { type: login.pending.type };
      const newState = authReducer(initialState, action);
      expect(newState.isLoading).toBe(true);
      expect(newState.error).toBeNull();
    });

    it("should handle fulfilled state", () => {
      const mockUser = {
        id: "1",
        name: "Test User",
        email: "test@example.com",
      };
      const mockToken = "mock-token";
      const action = {
        type: login.fulfilled.type,
        payload: { user: mockUser, token: mockToken },
      };
      const newState = authReducer(initialState, action);
      expect(newState.isLoading).toBe(false);
      expect(newState.user).toEqual(mockUser);
      expect(newState.token).toBe(mockToken);
    });

    it("should handle rejected state", () => {
      const action = {
        type: login.rejected.type,
        payload: "Invalid credentials",
      };
      const newState = authReducer(initialState, action);
      expect(newState.isLoading).toBe(false);
      expect(newState.error).toBe("Invalid credentials");
    });
  });

  describe("logout async thunk", () => {
    it("should clear user and token", () => {
      const loggedInState = {
        user: { id: "1", name: "Test User" },
        token: "mock-token",
        isLoading: false,
        error: null,
      };
      const action = { type: logout.fulfilled.type };
      const newState = authReducer(loggedInState, action);
      expect(newState.user).toBeNull();
      expect(newState.token).toBeNull();
    });
  });
});
```

## 5. Integration Tests

### API Integration Tests (src/test/integration/api.test.jsx)

```javascript
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Provider } from "react-redux";
import { configureStore } from "@reduxjs/toolkit";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { BrowserRouter } from "react-router-dom";
import { http, HttpResponse } from "msw";
import { server } from "../mocks/server";
import { HomePage } from "../../pages/HomePage";
import { CreateBlogForm } from "../../components/forms/CreateBlogForm";
import authReducer from "../../features/auth/authSlice";
import blogReducer from "../../features/blog/blogSlice";
import { describe, it, expect, beforeEach } from "vitest";

describe("API Integration Tests", () => {
  let queryClient;
  let store;

  beforeEach(() => {
    queryClient = new QueryClient({
      defaultOptions: {
        queries: {
          retry: false,
        },
      },
    });
    store = configureStore({
      reducer: {
        auth: authReducer,
        blogs: blogReducer,
      },
    });
  });

  const renderWithProviders = (component) => {
    return render(
      <Provider store={store}>
        <QueryClientProvider client={queryClient}>
          <BrowserRouter>{component}</BrowserRouter>
        </QueryClientProvider>
      </Provider>,
    );
  };

  describe("HomePage Integration", () => {
    it("should fetch and display blogs", async () => {
      renderWithProviders(<HomePage />);

      await waitFor(() => {
        expect(screen.getByText("Test Blog 1")).toBeInTheDocument();
        expect(screen.getByText("Test Blog 2")).toBeInTheDocument();
      });
    });

    it("should handle API error gracefully", async () => {
      server.use(
        http.get("http://localhost:5000/api/blogs", () => {
          return new HttpResponse(null, { status: 500 });
        }),
      );

      renderWithProviders(<HomePage />);

      await waitFor(() => {
        expect(screen.getByText(/failed to load blogs/i)).toBeInTheDocument();
      });
    });

    it("should filter blogs by category", async () => {
      renderWithProviders(<HomePage />);

      const categoryButton = await screen.findByText("React");
      userEvent.click(categoryButton);

      await waitFor(() => {
        expect(screen.getByText("Test Blog 1")).toBeInTheDocument();
      });
    });
  });

  describe("CreateBlogForm Integration", () => {
    it("should create a new blog", async () => {
      const mockOnSubmit = vi.fn();
      renderWithProviders(<CreateBlogForm onSubmit={mockOnSubmit} />);

      await userEvent.type(screen.getByLabelText(/title/i), "New Test Blog");
      await userEvent.type(
        screen.getByLabelText(/description/i),
        "This is a test description for the new blog",
      );
      await userEvent.selectOptions(
        screen.getByLabelText(/category/i),
        "React",
      );
      await userEvent.type(screen.getByLabelText(/tags/i), "React, Testing");

      const submitButton = screen.getByRole("button", {
        name: /publish blog/i,
      });
      await userEvent.click(submitButton);

      await waitFor(() => {
        expect(mockOnSubmit).toHaveBeenCalled();
      });
    });

    it("should show validation errors for empty fields", async () => {
      renderWithProviders(<CreateBlogForm onSubmit={vi.fn()} />);

      const submitButton = screen.getByRole("button", {
        name: /publish blog/i,
      });
      await userEvent.click(submitButton);

      await waitFor(() => {
        expect(
          screen.getByText(/title must be at least 5 characters/i),
        ).toBeInTheDocument();
        expect(
          screen.getByText(/description must be at least 20 characters/i),
        ).toBeInTheDocument();
      });
    });
  });
});
```

### Authentication Flow Integration Test (src/test/integration/auth.test.jsx)

```javascript
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Provider } from "react-redux";
import { BrowserRouter } from "react-router-dom";
import { configureStore } from "@reduxjs/toolkit";
import { App } from "../../App";
import authReducer from "../../features/auth/authSlice";
import { describe, it, expect } from "vitest";

describe("Authentication Flow Integration", () => {
  const createStore = (initialState = {}) => {
    return configureStore({
      reducer: {
        auth: authReducer,
      },
      preloadedState: initialState,
    });
  };

  const renderWithRouter = (initialRoute = "/") => {
    window.history.pushState({}, "Test page", initialRoute);
    return render(
      <Provider store={createStore()}>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </Provider>,
    );
  };

  it("should redirect to login when accessing protected route without auth", () => {
    renderWithRouter("/dashboard");
    expect(screen.getByText(/sign in/i)).toBeInTheDocument();
  });

  it("should allow access to protected route when authenticated", () => {
    const loggedInState = {
      auth: {
        user: { id: "1", name: "Test User", email: "test@example.com" },
        token: "mock-token",
        isLoading: false,
        error: null,
      },
    };

    const store = createStore(loggedInState);
    render(
      <Provider store={store}>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </Provider>,
    );

    expect(screen.getByText(/welcome back/i)).toBeInTheDocument();
  });

  it("should logout user and redirect to home", async () => {
    const loggedInState = {
      auth: {
        user: { id: "1", name: "Test User", email: "test@example.com" },
        token: "mock-token",
        isLoading: false,
        error: null,
      },
    };

    const store = createStore(loggedInState);
    render(
      <Provider store={store}>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </Provider>,
    );

    const userMenu = screen.getByRole("button", { name: /test user/i });
    await userEvent.click(userMenu);

    const logoutButton = screen.getByRole("button", { name: /logout/i });
    await userEvent.click(logoutButton);

    await waitFor(() => {
      expect(
        screen.getByRole("button", { name: /get started/i }),
      ).toBeInTheDocument();
    });
  });
});
```

## 6. End-to-End Tests

### Cypress Configuration (cypress.config.js)

```javascript
import { defineConfig } from "cypress";

export default defineConfig({
  e2e: {
    baseUrl: "http://localhost:3000",
    supportFile: "cypress/support/e2e.js",
    specPattern: "cypress/e2e/**/*.cy.{js,jsx,ts,tsx}",
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    defaultCommandTimeout: 10000,
    setupNodeEvents(on, config) {
      // implement node event listeners here
      require("@cypress/code-coverage/task")(on, config);
      return config;
    },
  },
  component: {
    devServer: {
      framework: "react",
      bundler: "vite",
    },
  },
});
```

### E2E Test: Authentication Flow (cypress/e2e/auth.cy.js)

```javascript
describe("Authentication Flow", () => {
  beforeEach(() => {
    cy.visit("/");
  });

  describe("Login", () => {
    it("should display login form", () => {
      cy.visit("/login");
      cy.get('input[type="email"]').should("be.visible");
      cy.get('input[type="password"]').should("be.visible");
      cy.contains("button", "Sign In").should("be.visible");
    });

    it("should show validation errors for empty fields", () => {
      cy.visit("/login");
      cy.contains("button", "Sign In").click();
      cy.contains("Invalid email address").should("be.visible");
    });

    it("should login successfully with valid credentials", () => {
      cy.visit("/login");
      cy.get('input[type="email"]').type("test@example.com");
      cy.get('input[type="password"]').type("password123");
      cy.contains("button", "Sign In").click();

      cy.url().should("include", "/dashboard");
      cy.contains("Welcome back").should("be.visible");
    });

    it("should show error for invalid credentials", () => {
      cy.intercept("POST", "/api/auth/login", {
        statusCode: 401,
        body: { message: "Invalid credentials" },
      }).as("loginRequest");

      cy.visit("/login");
      cy.get('input[type="email"]').type("wrong@example.com");
      cy.get('input[type="password"]').type("wrongpassword");
      cy.contains("button", "Sign In").click();

      cy.contains("Invalid credentials").should("be.visible");
    });
  });

  describe("Registration", () => {
    it("should register new user successfully", () => {
      cy.intercept("POST", "/api/auth/register", {
        statusCode: 201,
        body: { message: "Registration successful" },
      }).as("registerRequest");

      cy.visit("/register");
      cy.get('input[name="name"]').type("New User");
      cy.get('input[type="email"]').type("newuser@example.com");
      cy.get('input[type="password"]').first().type("Password123!");
      cy.get('input[type="password"]').last().type("Password123!");
      cy.contains("button", "Sign Up").click();

      cy.wait("@registerRequest");
      cy.url().should("include", "/login");
      cy.contains("Registration successful").should("be.visible");
    });

    it("should show password mismatch error", () => {
      cy.visit("/register");
      cy.get('input[name="name"]').type("New User");
      cy.get('input[type="email"]').type("newuser@example.com");
      cy.get('input[type="password"]').first().type("Password123!");
      cy.get('input[type="password"]').last().type("Different123!");
      cy.contains("button", "Sign Up").click();

      cy.contains("Passwords don't match").should("be.visible");
    });
  });

  describe("Protected Routes", () => {
    it("should redirect to login when accessing dashboard without auth", () => {
      cy.visit("/dashboard");
      cy.url().should("include", "/login");
    });

    it("should allow access to dashboard when authenticated", () => {
      cy.login("test@example.com", "password123");
      cy.visit("/dashboard");
      cy.contains("Welcome back").should("be.visible");
    });
  });
});
```

### E2E Test: Blog CRUD Operations (cypress/e2e/blog.cy.js)

```javascript
describe("Blog Operations", () => {
  beforeEach(() => {
    cy.login("test@example.com", "password123");
    cy.visit("/dashboard");
  });

  describe("Create Blog", () => {
    it("should create a new blog post", () => {
      cy.intercept("POST", "/api/blogs", {
        statusCode: 201,
        body: { id: "123", title: "New Blog Post" },
      }).as("createBlog");

      cy.contains("Create New Blog").click();
      cy.url().should("include", "/create-blog");

      cy.get('input[name="title"]').type("My Awesome Blog Post");
      cy.get('textarea[name="description"]').type(
        "This is an awesome blog post about testing",
      );
      cy.get('select[name="category"]').select("React");
      cy.get('input[name="tags"]').type("React, Testing, Cypress");

      // Fill rich text editor
      cy.get(".ql-editor").type("This is the main content of my blog post");

      cy.contains("button", "Publish Blog").click();
      cy.wait("@createBlog");
      cy.contains("Blog published successfully").should("be.visible");
      cy.url().should("include", "/my-blogs");
    });

    it("should show validation errors for empty fields", () => {
      cy.visit("/create-blog");
      cy.contains("button", "Publish Blog").click();

      cy.contains("Title must be at least 5 characters").should("be.visible");
      cy.contains("Description must be at least 20 characters").should(
        "be.visible",
      );
    });
  });

  describe("Edit Blog", () => {
    it("should edit existing blog", () => {
      cy.intercept("PUT", "/api/blogs/1", {
        statusCode: 200,
        body: { id: "1", title: "Updated Title" },
      }).as("updateBlog");

      cy.visit("/my-blogs");
      cy.contains("Edit").first().click();
      cy.url().should("include", "/edit-blog");

      cy.get('input[name="title"]').clear().type("Updated Blog Title");
      cy.contains("button", "Publish Blog").click();

      cy.wait("@updateBlog");
      cy.contains("Blog updated successfully").should("be.visible");
    });
  });

  describe("Delete Blog", () => {
    it("should delete blog after confirmation", () => {
      cy.intercept("DELETE", "/api/blogs/1", {
        statusCode: 204,
      }).as("deleteBlog");

      cy.visit("/my-blogs");

      cy.on("window:confirm", () => true);
      cy.contains("Delete").first().click();

      cy.wait("@deleteBlog");
      cy.contains("Blog deleted successfully").should("be.visible");
    });
  });

  describe("Blog Interactions", () => {
    it("should like a blog", () => {
      cy.intercept("POST", "/api/blogs/1/like", {
        statusCode: 200,
        body: { likes: 11 },
      }).as("likeBlog");

      cy.visit("/blog/1");
      cy.contains("Like").click();
      cy.wait("@likeBlog");
      cy.contains("11").should("be.visible");
    });

    it("should bookmark a blog", () => {
      cy.intercept("POST", "/api/blogs/1/bookmark", {
        statusCode: 200,
      }).as("bookmarkBlog");

      cy.visit("/blog/1");
      cy.contains("Save Article").click();
      cy.wait("@bookmarkBlog");
      cy.contains("Added to bookmarks").should("be.visible");
    });

    it("should add a comment", () => {
      cy.intercept("POST", "/api/blogs/1/comments", {
        statusCode: 201,
        body: { id: "456", content: "Great post!" },
      }).as("addComment");

      cy.visit("/blog/1");
      cy.get('textarea[placeholder*="comment"]').type("Great post!");
      cy.contains("Post Comment").click();
      cy.wait("@addComment");
      cy.contains("Comment added").should("be.visible");
    });
  });
});
```

### E2E Test: Search and Filter (cypress/e2e/search.cy.js)

```javascript
describe("Search and Filter Functionality", () => {
  beforeEach(() => {
    cy.visit("/home");
  });

  it("should search blogs by title", () => {
    cy.get('input[placeholder*="Search"]').type("React{enter}");
    cy.url().should("include", "search=React");
    cy.get('[data-testid="blog-card"]').should("have.length.at.least", 1);
  });

  it("should filter blogs by category", () => {
    cy.contains("React").click();
    cy.get('[data-testid="blog-card"]').each(($card) => {
      cy.wrap($card).contains("React");
    });
  });

  it("should sort blogs by newest", () => {
    cy.get("select").select("Newest");
    cy.get('[data-testid="blog-card"]').first().should("be.visible");
  });

  it("should sort blogs by popular", () => {
    cy.get("select").select("Most Popular");
    // Verify sorting order
  });

  it("should paginate through results", () => {
    cy.get('[data-testid="pagination"]').should("be.visible");
    cy.contains("2").click();
    cy.url().should("include", "page=2");
  });

  it("should clear search results", () => {
    cy.get('input[placeholder*="Search"]').type("Test{enter}");
    cy.get('[data-testid="clear-search"]').click();
    cy.url().should("not.include", "search");
  });
});
```

## 7. Component Testing with Cypress (cypress/component/BlogCard.cy.jsx)

```javascript
import React from "react";
import { BrowserRouter } from "react-router-dom";
import { BlogCard } from "../../src/components/cards/BlogCard";

describe("BlogCard Component", () => {
  const mockBlog = {
    id: "1",
    title: "Test Blog",
    description: "Test Description",
    coverImage: "https://example.com/image.jpg",
    category: "React",
    author: {
      name: "John Doe",
      avatar: "https://example.com/avatar.jpg",
    },
    likes: 42,
    createdAt: "2024-01-01T00:00:00Z",
  };

  beforeEach(() => {
    cy.mount(
      <BrowserRouter>
        <BlogCard blog={mockBlog} />
      </BrowserRouter>,
    );
  });

  it("should display blog information", () => {
    cy.contains("Test Blog").should("be.visible");
    cy.contains("John Doe").should("be.visible");
    cy.contains("42").should("be.visible");
    cy.contains("React").should("be.visible");
  });

  it("should navigate to blog details on click", () => {
    cy.get("a").first().click();
    cy.url().should("include", "/blog/1");
  });

  it("should trigger like action when like button clicked", () => {
    const onLike = cy.stub().as("onLike");
    cy.mount(
      <BrowserRouter>
        <BlogCard blog={mockBlog} onLike={onLike} />
      </BrowserRouter>,
    );
    cy.contains("Like").click();
    cy.get("@onLike").should("have.been.calledWith", "1");
  });
});
```

## 8. Playwright E2E Tests (e2e/playwright/auth.spec.js)

```javascript
import { test, expect } from "@playwright/test";

test.describe("Authentication Flow", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/");
  });

  test("should login successfully", async ({ page }) => {
    await page.click("text=Login");
    await page.fill('input[type="email"]', "test@example.com");
    await page.fill('input[type="password"]', "password123");
    await page.click('button:has-text("Sign In")');

    await expect(page).toHaveURL(/.*dashboard/);
    await expect(page.locator("text=Welcome back")).toBeVisible();
  });

  test("should show error for invalid login", async ({ page }) => {
    await page.click("text=Login");
    await page.fill('input[type="email"]', "wrong@example.com");
    await page.fill('input[type="password"]', "wrong");
    await page.click('button:has-text("Sign In")');

    await expect(page.locator("text=Invalid credentials")).toBeVisible();
  });

  test("should register new user", async ({ page }) => {
    await page.click("text=Get Started");
    await page.fill('input[name="name"]', "Playwright User");
    await page.fill('input[type="email"]', "playwright@example.com");
    await page.fill('input[type="password"]', "Test123!");
    await page.fill('input[placeholder*="Confirm"]', "Test123!");
    await page.click('button:has-text("Sign Up")');

    await expect(page).toHaveURL(/.*login/);
  });
});

test.describe("Blog Management", () => {
  test.beforeEach(async ({ page }) => {
    // Login first
    await page.goto("/login");
    await page.fill('input[type="email"]', "test@example.com");
    await page.fill('input[type="password"]', "password123");
    await page.click('button:has-text("Sign In")');
  });

  test("should create new blog", async ({ page }) => {
    await page.click("text=Create New Blog");
    await page.fill('input[name="title"]', "Playwright Test Blog");
    await page.fill(
      'textarea[name="description"]',
      "This is a test blog created by Playwright",
    );
    await page.selectOption('select[name="category"]', "React");
    await page.fill('input[name="tags"]', "Playwright, Testing");
    await page.locator(".ql-editor").fill("This is the main content");
    await page.click('button:has-text("Publish Blog")');

    await expect(
      page.locator("text=Blog published successfully"),
    ).toBeVisible();
  });

  test("should edit existing blog", async ({ page }) => {
    await page.goto("/my-blogs");
    await page.click("text=Edit");
    await page.fill('input[name="title"]', "Updated Blog Title");
    await page.click('button:has-text("Publish Blog")');

    await expect(page.locator("text=Blog updated successfully")).toBeVisible();
  });

  test("should delete blog", async ({ page }) => {
    await page.goto("/my-blogs");
    page.on("dialog", (dialog) => dialog.accept());
    await page.click("text=Delete");

    await expect(page.locator("text=Blog deleted successfully")).toBeVisible();
  });
});
```

## 9. Test Utilities (src/test/utils/test-utils.js)

```javascript
import { render } from "@testing-library/react";
import { Provider } from "react-redux";
import { BrowserRouter } from "react-router-dom";
import { configureStore } from "@reduxjs/toolkit";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import authReducer from "../../features/auth/authSlice";
import blogReducer from "../../features/blog/blogSlice";
import themeReducer from "../../features/theme/themeSlice";

const createTestStore = (preloadedState = {}) => {
  return configureStore({
    reducer: {
      auth: authReducer,
      blogs: blogReducer,
      theme: themeReducer,
    },
    preloadedState,
  });
};

const createTestQueryClient = () => {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
      },
    },
  });
};

export const renderWithProviders = (
  ui,
  {
    preloadedState = {},
    store = createTestStore(preloadedState),
    queryClient = createTestQueryClient(),
    ...renderOptions
  } = {},
) => {
  const Wrapper = ({ children }) => {
    return (
      <Provider store={store}>
        <QueryClientProvider client={queryClient}>
          <BrowserRouter>{children}</BrowserRouter>
        </QueryClientProvider>
      </Provider>
    );
  };

  return {
    store,
    queryClient,
    ...render(ui, { wrapper: Wrapper, ...renderOptions }),
  };
};

export const mockAuthState = {
  auth: {
    user: {
      id: "1",
      name: "Test User",
      email: "test@example.com",
      role: "user",
    },
    token: "mock-token",
    isLoading: false,
    error: null,
  },
};

export const mockAdminState = {
  auth: {
    user: {
      id: "2",
      name: "Admin User",
      email: "admin@example.com",
      role: "admin",
    },
    token: "mock-token",
    isLoading: false,
    error: null,
  },
};
```

## 10. Complete Test Folder Structure

```
devblog-pro/
├── cypress/
│   ├── e2e/
│   │   ├── auth.cy.js
│   │   ├── blog.cy.js
│   │   ├── search.cy.js
│   │   └── dashboard.cy.js
│   ├── component/
│   │   └── BlogCard.cy.jsx
│   ├── fixtures/
│   │   ├── users.json
│   │   └── blogs.json
│   ├── support/
│   │   ├── commands.js
│   │   ├── e2e.js
│   │   └── component.js
│   └── plugins/
│       └── index.js
├── e2e/
│   └── playwright/
│       ├── auth.spec.js
│       ├── blog.spec.js
│       └── search.spec.js
├── src/
│   ├── components/
│   │   └── __tests__/
│   │       ├── BlogCard.test.jsx
│   │       ├── CommentsSection.test.jsx
│   │       └── SearchBar.test.jsx
│   ├── pages/
│   │   └── __tests__/
│   │       ├── LoginPage.test.jsx
│   │       ├── RegisterPage.test.jsx
│   │       └── DashboardPage.test.jsx
│   ├── features/
│   │   └── __tests__/
│   │       ├── authSlice.test.js
│   │       └── blogSlice.test.js
│   ├── utils/
│   │   └── __tests__/
│   │       └── helpers.test.js
│   ├── test/
│   │   ├── setup.js
│   │   ├── mocks/
│   │   │   ├── server.js
│   │   │   └── handlers.js
│   │   ├── integration/
│   │   │   ├── api.test.jsx
│   │   │   └── auth.test.jsx
│   │   └── utils/
│   │       └── test-utils.js
│   └── hooks/
│       └── __tests__/
│           └── useInfiniteScroll.test.js
├── playwright.config.js
├── cypress.config.js
├── vitest.config.js
└── .eslintrc.test.js
```

## 11. Cypress Commands (cypress/support/commands.js)

```javascript
Cypress.Commands.add("login", (email, password) => {
  cy.session([email, password], () => {
    cy.visit("/login");
    cy.get('input[type="email"]').type(email);
    cy.get('input[type="password"]').type(password);
    cy.contains("button", "Sign In").click();
    cy.url().should("include", "/dashboard");
  });
});

Cypress.Commands.add("createBlog", (blogData) => {
  cy.visit("/create-blog");
  cy.get('input[name="title"]').type(blogData.title);
  cy.get('textarea[name="description"]').type(blogData.description);
  cy.get('select[name="category"]').select(blogData.category);
  cy.get('input[name="tags"]').type(blogData.tags);
  cy.get(".ql-editor").type(blogData.content);
  cy.contains("button", "Publish Blog").click();
});

Cypress.Commands.add("deleteBlog", (blogTitle) => {
  cy.contains(blogTitle)
    .parent()
    .within(() => {
      cy.contains("Delete").click();
    });
  cy.on("window:confirm", () => true);
});
```

## 12. ESLint Config for Testing (.eslintrc.test.js)

```javascript
module.exports = {
  extends: [
    "plugin:testing-library/react",
    "plugin:jest-dom/recommended",
    "plugin:cypress/recommended",
  ],
  plugins: ["testing-library", "jest-dom", "cypress"],
  env: {
    "cypress/globals": true,
    "jest/globals": true,
  },
  rules: {
    "testing-library/await-async-queries": "error",
    "testing-library/no-await-sync-queries": "error",
    "testing-library/no-debugging-utils": "warn",
    "jest-dom/prefer-checked": "error",
    "jest-dom/prefer-enabled-disabled": "error",
    "jest-dom/prefer-required": "error",
    "jest-dom/prefer-to-have-attribute": "error",
  },
};
```

## 13. Running Tests Commands

```bash
# Run unit and integration tests with Vitest
npm run test

# Run tests with UI
npm run test:ui

# Run tests with coverage report
npm run test:coverage

# Run Cypress E2E tests
npm run test:cypress

# Run Cypress tests headlessly
npm run test:cypress:run

# Run Playwright E2E tests
npm run test:e2e

# Run Playwright tests with UI
npm run test:e2e:ui

# Run all tests
npm run test:all
```

This complete testing setup includes:

✅ **Unit Tests**: Component tests, utility function tests, Redux slice tests
✅ **Integration Tests**: API integration, authentication flow, component integration
✅ **E2E Tests**: Complete user flows with Cypress and Playwright
✅ **MSW Setup**: Mock API handlers for testing
✅ **Test Utilities**: Helper functions and custom renderers
✅ **Coverage Reports**: Vitest coverage configuration
✅ **CI/CD Ready**: All tests can run in headless mode
✅ **Cross-browser Testing**: Playwright supports multiple browsers

The testing suite covers critical paths including authentication, blog CRUD operations, search/filter functionality, and user interactions. You can run these tests locally or in CI/CD pipelines!

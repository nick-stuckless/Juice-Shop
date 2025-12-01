# Juice-Shop
This repository is designed to aid you in the setup and configuration of a testing environment for use with OWASP Juice Shop.
# OWASP Juice Shop SQL Injection Lab #

This lab demonstrates how to identify and mitigate a classic SQL Injection vulnerability in the OWASP Juice Shop application. The goal is to show both the exploitation of the flaw and the application of a secure coding fix, packaged into a Docker container for easy deployment. The objectives are to run Juice Shop in a containerized environment, exploit the vulnerable login endpoint using SQL injection, apply a code-level fix to prevent injection, rebuild and run a custom Docker image with the mitigation applied, and validate the fix with before/after testing evidence.

The original login logic in `routes/login.ts` concatenated user input directly into a SQL query:
models.sequelize.query(`SELECT * FROM Users WHERE email = '${req.body.email}' AND password = '${security.hash(req.body.password)}' AND deletedAt IS NULL`, { model: UserModel, plain: true })
This allowed attackers to bypass authentication with payloads such as `' OR 1=1--`.

The vulnerable query was replaced with a parameterized query using Sequelize’s `replacements`:
models.sequelize.query(
  "SELECT * FROM Users WHERE email = ? AND password = ? AND deletedAt IS NULL",
  {
    replacements: [req.body.email || '', security.hash(req.body.password || '')],
    model: UserModel,
    plain: true
  }
)
This ensures user input is treated strictly as data, preventing malicious SQL execution.

To build and run the lab in Docker, clone the repository and build the custom image with:
docker build -t my-juice-shop .
Then run the container:
docker run -p 3000:3000 my-juice-shop
Juice Shop will be available at http://localhost:3000.

Validation is done by testing the exploit before and after the fix. Before the fix:
curl -X POST http://localhost:3000/rest/user/login -H "Content-Type: application/json" -d '{"email":"\' OR 1=1--","password":"anything"}'
Result: Authentication succeeds, token returned. After the fix:
curl -X POST http://localhost:3000/rest/user/login -H "Content-Type: application/json" -d '{"email":"\' OR 1=1--","password":"anything"}'
Result: 401 Unauthorized – Invalid email or password.

For detailed instructions, see the attached reports and guides in this repository. This lab highlights the importance of secure coding practices in database interactions. By replacing unsafe string concatenation with parameterized queries, SQL injection attacks are effectively prevented while maintaining normal application functionality.


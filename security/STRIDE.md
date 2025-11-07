**STRIDE** is a widely used threat modeling framework developed by Microsoft to help identify potential security threats in software systems. Each letter in STRIDE represents a different category of threat. It’s commonly used in Azure architecture and application security to systematically think about risks during design or review.

| Letter | Meaning                     | Description                                           | Example in Web/App context                                                               |
| ------ | --------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **S**  | **Spoofing**                | Pretending to be someone else (identity theft).       | An attacker uses stolen credentials or forges a JWT token to access another user’s data. |
| **T**  | **Tampering**               | Unauthorized modification of data.                    | Changing JSON payloads in transit to escalate privileges or modify order amounts.        |
| **R**  | **Repudiation**             | Denial of actions or transactions.                    | A user denies sending a request; without proper logging, you can’t prove it happened.    |
| **I**  | **Information Disclosure**  | Unauthorized access to sensitive information.         | Exposing PII, API keys, or passwords through logs or APIs.                               |
| **D**  | **Denial of Service (DoS)** | Making a system unavailable or degrading performance. | Flooding an API with requests until it crashes or becomes unresponsive.                  |
| **E**  | **Elevation of Privilege**  | Gaining higher access than allowed.                   | Exploiting a bug to gain admin rights from a normal user account.                        |



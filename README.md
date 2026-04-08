# Password-Strength-Checker
import string
from getpass import getpass
from typing import List, Tuple

def check_password(password: str, min_length: int = 8) -> Tuple[str, List[str], int]:
    """
    Check password strength and return status with missing requirements.

    Returns:
        Tuple of (strength_label, missing_requirements, score)
    """
    missing = []

    # Length check
    if len(password) < min_length:
        missing.append(f"at least {min_length} characters (current: {len(password)})")

    # Character type checks
    if not any(c.isupper() for c in password):
        missing.append("one uppercase letter")

    if not any(c.islower() for c in password):
        missing.append("one lowercase letter")

    if not any(c.isdigit() for c in password):
        missing.append("one digit")

    if not any(c in string.punctuation for c in password):
        missing.append("one special character")

    # Calculate score (0-100)
    score = max(0, 100 - (len(missing) * 20))

    # Determine strength
    if len(missing) == 0:
        strength = "Strong"
    elif len(missing) <= 2:
        strength = "Medium"
    else:
        strength = "Weak"

    return strength, missing, score

def main():
    try:
        # getpass hides the password as you type
        password = getpass("Enter password: ")

        # Confirm password if creating new account (optional)
        # confirm = getpass("Confirm password: ")
        # if password != confirm:
        #     print("Passwords don't match!")
        #     return

        strength, missing, score = check_password(password)

        print(f"\n{'='*40}")
        print(f"Password Status: {strength} ({score}/100)")
        print(f"Length: {len(password)} characters")
        print(f"{'='*40}")

        if missing:
            print(f"\nMissing requirements ({len(missing)} of 5):")
            for i, req in enumerate(missing, 1):
                print(f"  {i}. {req}")
        else:
            print("\n✓ All security conditions satisfied!")
            if len(password) >= 12:
                print("✓ Excellent length (12+ characters)")

    except KeyboardInterrupt:
        print("\n\nOperation cancelled.")

if __name__ == "__main__":
    main()
```

## Alternative: Weighted Scoring System

If you prefer scoring based on **security importance** (length matters more than special chars):

```python
def check_password_weighted(password: str) -> dict:
    checks = {
        'length': (len(password) >= 8, 30),
        'length_12': (len(password) >= 12, 10),  # bonus
        'uppercase': (any(c.isupper() for c in password), 15),
        'lowercase': (any(c.islower() for c in password), 15),
        'digit': (any(c.isdigit() for c in password), 15),
        'special': (any(c in string.punctuation for c in password), 15),
    }

    score = sum(weight for passed, weight in checks.values() if passed)

    if score >= 90:
        strength = "Very Strong"
    elif score >= 70:
        strength = "Strong"
    elif score >= 40:
        strength = "Medium"
    else:
        strength = "Weak"

    return {
        'strength': strength,
        'score': score,
        'passed': [k for k, (v, _) in checks.items() if v],
        'failed': [k for k, (v, _) in checks.items() if not v]
    }

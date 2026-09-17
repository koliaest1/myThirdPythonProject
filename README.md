# Waracle QA Technical Exercise — Mobile Test Suite

Automated end-to-end and negative test suite built for the Waracle QA Technical Exercise using Maestro and Expo.

## Tech Stack & Tools
* Test Framework: Maestro (YAML-based UI testing)
* Target Application: Expo Mobile App (`host.exp.exponent` / `host.exp.Exponent`)
* Language: YAML

---

## Repository Structure
```text
.
├── tests/
│   ├── ios/
│   │   ├── happy_path.yaml
│   │   └── negative_edge_case.yaml
│   └── android/
│       ├── happy_path.yaml
│       └── negative_edge_case.yaml
└── README.md
```


## How to Run the Tests
1. Ensure you have Maestro installed on your machine
```bash
curl -Ls "[https://get.maestro.mobile.dev](https://get.maestro.mobile.dev)" | bash
export PATH="$HOME/.maestro/bin:$PATH"
echo 'export PATH="$HOME/.maestro/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```
2. Start your mobile emulator/simulator and ensure the app is installed and active.
3. Run the test suites from the root directory depending on your target platform:
  For iOS:
```bash
maestro test tests/ios/happy_path.yaml
```
```bash
maestro test tests/ios/negative_edge_case.yaml
```
  for Android:
```bash
maestro test tests/android/happy_path.yaml
```
```bash
maestro test tests/android/negative_edge_case.yaml
```


## QA Release Notes, Observations & Concerns
1. Critical Bug Found (AC-2 Violation)
    Title: The promo code applies wrong discount
    Description:
        STR:
            - Open Cart and tap Coupon input field
            - Enter "WARACLE25" and tap "Apply"
        AR:
            - Discount of £0.25 applied
        ER:
            - Discount equal to 25% of cart Subtotal is applied

    Impact: High. Customers are short-changed on their expected promotional discount, risking cart abandonment and user trust.

2. Additional QA Observations & Recommended Edge-Case Enhancements:
While not explicitly requested in the baseline acceptance criteria, a robust production release should also account for the following edge cases:
- Ideally the 'Apply' button should remain disabled when the coupon input field is empty to prevent unnecessary invalid state errors.
```YAML
  # 2. Negative Test: Empty Code Submission (my own vision of behavior: button should be disabled when the input is empty)
    - tapOn:
        id: "cart_coupon_input"
    - eraseText
    - hideKeyboard
    - assertVisible:
        id: "cart_apply_button"
        enabled: false # Assert that "Apply" button is disabled when no text is entered in the input field
```
- Case Sensitivity: Ensure promo code validation handles lowercase input (e.g., waracle25) gracefully without forcing user frustration.
```YAML
  # 4. Edge Case: input is case-sensitive (not applicable to current app but might be useful for a real one)
    - tapOn:
        id: "cart_coupon_input"
    - eraseText
    - inputText: "waracle25"
    - hideKeyboard
    - tapOn:
        id: "cart_apply_button"
    - assertVisible: "That coupon is not valid." # Assert that a clear error message
    - assertVisible: "£29.99" #Verification that total remains unchanged
```
- Duplicate/Stacked Applications: Implement validation to prevent users from applying the same active code multiple times (e.g., throwing a "Coupon already applied" warning).
```YAML
# 5. Edge Case: Applying the same coupon twice (not applicable to current app but might be useful for a real one)
    - tapOn:
        id: "cart_coupon_input"
    - eraseText
    - inputText: "WARACLE25"
    - hideKeyboard
    - tapOn:
        id: "cart_apply_button"
    - assertVisible: "Discount (WARACLE25)"
    - tapOn:
        id: "cart_coupon_input"
    - eraseText
    - inputText: "WARACLE25"
    - hideKeyboard
    - assertVisible: "Coupon already applied"
    - assertVisible: "29.74" #Verification that total remains unchanged
```

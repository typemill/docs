the current input form for the 2fa authentication / authentication code provides six input fields for 6 numbers.

This is quite annoying for users because they cannot easily copy and paste the six numbers at once.

Please change the input field and the validation so it is just a simple input field that accepts 6 numbers from 0-9.

Affected files are at least:

* /system/typemill/author/auth/authcode.twig
* /system/typemill/Controllers/ControllerWebAuth.php
* /system/typemill/Models/Validation.php

and maybe more.

---

## Implementation Plan

### 1. `/system/typemill/author/auth/authcode.twig`
- Replace the 6 separate `<input>` fields (`code-1` … `code-6`) with a single `<input>` field named `authcode`.
- Attributes: `type="text"`, `inputmode="numeric"`, `name="authcode"`, `pattern="[0-9]{6}"`, `maxlength="6"`, `required`.
- Remove the inline `<script>` and `moveToNextField()` function.
- Keep existing styling classes for visual consistency.

### 2. `/system/typemill/Models/Validation.php`
- Update `authcode()` validation method:
  - Change required fields from `['username', 'code-1', …, 'code-6']` to `['username', 'authcode']`.
  - Replace the six per-digit `regex` rules with one rule on `authcode`: `/^[0-9]{6}$/`.

### 3. `/system/typemill/Controllers/ControllerWebAuth.php`
- In `loginWithAuthcode()`, replace the concatenation line:
  ```php
  $authcodevalue = $input['code-1'] . $input['code-2'] . $input['code-3'] . $input['code-4'] . $input['code-5'] . $input['code-6'];
  ```
  with:
  ```php
  $authcodevalue = $input['authcode'] ?? '';
  ```

### 4. Verification
- Confirm the form renders a single input.
- Confirm copy-paste of a 6-digit code works.
- Confirm validation rejects non-numeric or wrong-length input.
- Confirm successful login with a valid code still works.

**Awaiting confirmation before implementation.**
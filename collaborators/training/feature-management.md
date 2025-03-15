# Introduction to Feature Management Using Flags

When building software for your school project, you might want to add new features or test changes without disrupting the experience for everyone using your application. This is where **feature flags** come in handy. Feature flags (also called feature toggles) are tools that let you turn specific features on or off without changing your code. They give you control over how and when features are released.

## Why Are Feature Flags Necessary?

Imagine you're creating a project for your school, like a website for managing club activities. You decide to add a new feature that allows students to vote for club leaders. Without feature flags, you might release this feature to everyone at once. But what if:

- The feature has a bug that prevents voting from working?
- You want to test the feature with just a small group of students first?
- The feature isn't ready, but you accidentally release it anyway?

In any of these cases, your project could break or confuse users. Feature flags help you avoid these problems by letting you control who sees the feature and when.

## Examples of What Can Go Wrong Without Feature Flags

1. **Unintended Bugs:** A new feature might crash the entire application if it’s not fully tested.
2. **Incomplete Features:** Users might see unfinished features that don’t work as expected.
3. **No Rollback Option:** If something goes wrong, you’d have to rewrite or remove code to fix it, which can take time.

## How Are Feature Flags Implemented?

Feature flags can be implemented in different ways, depending on your project. Here are some common methods:

1. **Using Conditional Statements:**  
    In your code, you can use `if` statements to check whether a feature should be enabled. For example:  
    ```python
    enable_voting_feature = True

    if enable_voting_feature:
        print("Show voting page")
    else:
        print("Show default page")
    ```

2. **Configuration Files:**  
    Store feature flags in a configuration file that your application reads. This makes it easier to update flags without changing the code.  
    Example `config.json`:  
    ```json
    {
        "enable_voting_feature": true
    }
    ```
    Example Python code to read the configuration:  
    ```python
    import json

    with open("config.json", "r") as config_file:
        config = json.load(config_file)

    if config.get("enable_voting_feature"):
        print("Show voting page")
    else:
        print("Show default page")
    ```

3. **Simple Flag Management in Code:**  
    For small projects, you can manage feature flags directly in your code using dictionaries:  
    ```python
    feature_flags = {
        "enable_voting_feature": True
    }

    if feature_flags["enable_voting_feature"]:
        print("Show voting page")
    else:
        print("Show default page")
    ```

## Conclusion

Feature flags are a powerful way to manage changes in your project. They let you test new features safely, control who sees them, and quickly turn them off if something goes wrong. By using feature flags, you can ensure your school project runs smoothly while you continue to improve it.  

## Further reading

- [Feature Toggles](https://martinfowler.com/articles/feature-toggles.html) (code snippets in javascript)

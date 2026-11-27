# Contributing to MyVote AWS

First off, thank you for considering contributing to MyVote AWS! It's people like you that make this project better.

## 🤝 Code of Conduct

This project and everyone participating in it is governed by our Code of Conduct. By participating, you are expected to uphold this code.

## 🐛 How to Report a Bug

If you find a bug, please create an issue on GitHub with the following information:

### Bug Report Template
```
**Describe the bug**
A clear and concise description of what the bug is.

**To Reproduce**
Steps to reproduce the behavior:
1. Go to '...'
2. Click on '....'
3. See error

**Expected behavior**
A clear description of what you expected to happen.

**Screenshots**
If applicable, add screenshots to help explain your problem.

**Environment:**
 - AWS Region: [e.g., us-east-1]
 - Browser [e.g., chrome, safari]
 - Deployment method [e.g., Terraform, CloudFormation]

**Additional context**
Add any other context about the problem here.
```

## 💡 How to Suggest a Feature

Feature requests are welcome! Please create an issue on GitHub with:

### Feature Request Template
```
**Is your feature request related to a problem?**
A clear description of what the problem is.

**Describe the solution you'd like**
A clear description of what you want to happen.

**Describe alternatives you've considered**
Any alternative solutions or features you've considered.

**Additional context**
Add any other context or screenshots about the feature request.
```

## 🔧 Development Setup

1. **Fork and Clone**
   ```bash
   git clone https://github.com/YOUR_USERNAME/My-Vote.git
   cd My-Vote
   ```

2. **Set up AWS Credentials**
   ```bash
   aws configure
   ```

3. **Install Dependencies**
   ```bash
   # For Python components
   pip install boto3 pytest

   # For local testing
   pip install moto  # AWS service mocking
   ```

4. **Create a Feature Branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

## 📝 Pull Request Process

1. **Update Documentation**
   - Update the README.md with details of changes if needed
   - Update component-specific READMEs
   - Add comments to complex code

2. **Follow Code Style**
   - Python: Follow PEP 8 guidelines
   - JavaScript: Use consistent formatting
   - Terraform: Follow HashiCorp style guide

3. **Test Your Changes**
   ```bash
   # Test Python code
   python -m pytest tests/

   # Test Terraform
   cd iac
   terraform validate
   terraform plan
   ```

4. **Commit with Clear Messages**
   ```bash
   git commit -m "feat: add new voting option feature"
   git commit -m "fix: resolve DynamoDB connection issue"
   git commit -m "docs: update deployment instructions"
   ```

   Use conventional commit prefixes:
   - `feat:` New features
   - `fix:` Bug fixes
   - `docs:` Documentation changes
   - `style:` Code style changes
   - `refactor:` Code refactoring
   - `test:` Test additions or changes
   - `chore:` Build process or tooling changes

5. **Push and Create PR**
   ```bash
   git push origin feature/your-feature-name
   ```
   Then create a Pull Request on GitHub

6. **PR Requirements**
   - Clear title and description
   - Link to related issues
   - All CI checks passing
   - At least one maintainer approval

## 🧪 Testing Guidelines

### Unit Tests
```python
# Example test structure
import pytest
from voting_backend import lambda_handler

def test_lambda_handler_success():
    event = {
        'body': '{"vote": "a", "voter": "test"}'
    }
    response = lambda_handler(event, None)
    assert response['statusCode'] == 200
```

### Integration Tests
- Test API Gateway endpoints
- Verify Lambda-DynamoDB integration
- Check SQS message processing

### Local Testing
```bash
# Run voting backend locally
cd voting-backend
python -m pytest test_voting.py

# Run with AWS mocking
python -m pytest --moto
```

## 📋 Coding Standards

### Python
```python
# Good
def process_vote(vote_data: dict) -> dict:
    """
    Process a vote and store in DynamoDB.
    
    Args:
        vote_data: Dictionary containing vote information
        
    Returns:
        dict: Processing result with status and message
    """
    # Implementation
    pass

# Use type hints
# Add docstrings
# Handle exceptions properly
# Use logging instead of print()
```

### JavaScript
```javascript
// Good
function submitVote(voteValue) {
    // Use meaningful variable names
    const voteData = {
        MessageAttributes: {
            vote: { Type: "String", StringValue: voteValue }
        }
    };
    
    // Handle errors properly
    return $.post(backendUrl, JSON.stringify(voteData))
        .done(handleSuccess)
        .fail(handleError);
}
```

### Terraform
```hcl
# Good
resource "aws_lambda_function" "voting_backend" {
  function_name = "${var.project_name}-voting-backend"
  handler       = "voting.lambda_handler"
  runtime       = "python3.9"
  
  # Use variables
  # Add comments for complex resources
  # Follow naming conventions
  
  tags = {
    Name        = "Voting Backend Lambda"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

## 🏗️ Architecture Changes

For significant architectural changes:

1. **Create an RFC (Request for Comments)**
   - Describe the problem
   - Propose solution with diagrams
   - List pros and cons
   - Estimate cost impact

2. **Discuss in Issues**
   - Create an issue with `[RFC]` prefix
   - Wait for community feedback
   - Iterate on the design

3. **Update Documentation**
   - Architecture diagrams
   - Component flow charts
   - Cost estimates

## 🔍 Code Review Process

### As a Contributor
- Respond to feedback promptly
- Make requested changes
- Keep PR focused and small
- Be open to suggestions

### As a Reviewer
- Be respectful and constructive
- Test the changes locally
- Check for security issues
- Verify documentation updates

## 📚 Additional Resources

- [AWS Best Practices](https://aws.amazon.com/architecture/well-architected/)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [Python Best Practices](https://docs.python-guide.org/)

## 🎯 Areas for Contribution

We especially welcome contributions in:

- 🧪 **Testing**: Add unit and integration tests
- 📖 **Documentation**: Improve guides and examples
- 🔒 **Security**: Enhance security configurations
- 💰 **Cost Optimization**: Reduce AWS costs
- 🎨 **UI/UX**: Improve frontend design
- 🚀 **Performance**: Optimize Lambda functions
- 🔧 **DevOps**: Add CI/CD pipelines
- 🌐 **Localization**: Add multi-language support

## 📮 Contact

Questions about contributing? Open an issue with the `question` label!

---

Thank you for contributing to MyVote AWS! 🎉

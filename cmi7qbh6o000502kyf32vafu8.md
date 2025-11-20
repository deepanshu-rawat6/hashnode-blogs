---
title: "Creating and Hosting a Jenkins Plugin: A Complete Guide"
seoTitle: "How to Build & Publish Your Jenkins Plugin"
seoDescription: "Need to extend Jenkins functionality? Follow our guide to design, code, and publish your own custom Jenkins plugins using Java and Apache Maven."
datePublished: Thu Nov 20 2025 17:52:40 GMT+0000 (Coordinated Universal Time)
cuid: cmi7qbh6o000502kyf32vafu8
slug: creating-and-hosting-a-jenkins-plugin-a-complete-guide
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1763660813469/69d0c493-2077-4969-abf0-5eeb93775e88.png

---

# Introduction

Jenkins is one of the most popular automation servers in the world, powering CI/CD pipelines for organizations of all sizes. While Jenkins offers extensive out-of-the-box functionality, its true power lies in its extensibility through plugins. With over 1,800 plugins available in the Jenkins ecosystem, there’s a plugin for almost every use case—but what happens when you need something that doesn’t exist yet? That’s where custom plugin development comes in. Whether you’re integrating a proprietary tool, enforcing organizational policies, or solving a unique automation challenge, creating a Jenkins plugin allows you to extend Jenkins to meet your specific needs. This guide will walk you through the entire journey—from setting up your development environment to publishing your plugin in the Jenkins Marketplace, making it available to the global Jenkins community.

![sherlock-jenkins](https://cdn.hashnode.com/res/hashnode/image/upload/v1763659552568/585bd019-6bf9-4e13-af6a-c594be77d5c0.png align="center")

---

# Why Create a Jenkins Plugin?

Before diving into the technical details, let’s explore some common reasons for creating a custom Jenkins plugin:

* **Integration**: Connect Jenkins with proprietary or third-party tools that don’t have existing plugin
    

* **Custom Workflows**: Implement organization-specific build steps or validation logic
    
* **Security and Compliance**: Enforce security gates, policy checks, or compliance requirements in your CI/CD pipeline
    
* **Automation**: Automate repetitive tasks or complex orchestration scenarios
    
* **Community Contribution**: Share solutions with the Jenkins community and contribute to the open-source ecosystem
    

---

# Prerequisites and Environment Setup

Before you begin developing your plugin, ensure you have the necessary tools and environment configured.

## Required Software

### Java Development Kit (JDK)

Jenkins plugin development requires **Java 21**, which is the recommended version for Jenkins users. You can download it from:

* [Eclipse Temurin](https://adoptium.net/)
    
* Your Linux distribution’s package manager
    

Verify your Java installation:

```bash
java -version
```

### Apache Maven

Jenkins plugins are built using **Apache Maven 3.9.6** **or newer**. Using an older version may result in errors like “Unknown packaging: hpi.”

1. Download Maven from the [Apache Maven website](https://maven.apache.org/download.cgi)
    
2. Extract the archive to your desired location (do not move files after extraction)
    
3. Add Maven’s bin/ directory to your system’s PATH environment variable
    

Verify your Maven installation:

```bash
mvn -version
```

You should see both Java 21 and Maven version information displayed.

## Maven Configuration

Create or update the `settings.xml` file in your `~/.m2` directory (or `%USERPROFILE%.m2\settings.xml` on Windows):

```xml
<settings>
  <pluginGroups>
    <pluginGroup>org.jenkins-ci.tools</pluginGroup>
  </pluginGroups>

  <profiles>
    <profile>
      <id>jenkins</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>repo.jenkins-ci.org</id>
          <url>https://repo.jenkins-ci.org/public/</url>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>repo.jenkins-ci.org</id>
          <url>https://repo.jenkins-ci.org/public/</url>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
</settings>
```

## IDE Setup (Optional)

While not required, using an IDE can significantly enhance your development experience. Popular choices include:

* **IntelliJ IDEA**: Offers excellent Maven support and Jenkins-specific plugins
    
* **Eclipse**: Includes native Maven integration
    
* **NetBeans**: Provides built-in Maven support
    

All three IDEs have specialized plugins for Jenkins development that provide additional features like syntax highlighting for Jelly files and easy navigation of extension points.

---

# Creating Your First Plugin

With your environment set up, you’re ready to create your first Jenkins plugin.

### Generate the Plugin Project

Jenkins provides Maven archetypes that generate plugin project scaffolding. Run this command in your desired directory:

```bash
mvn -U archetype:generate -Dfilter=io.jenkins.archetypes:
```

You’ll be presented with several archetype options:

1. **empty-plugin**: A minimal plugin skeleton
    
2. **hello-world-plugin**: Includes a sample build step (recommended for beginners)
    
3. **scripted-pipeline**: For pipeline DSL plugins
    
4. **global-configuration-plugin**: For plugins with global configuration
    

For this tutorial, select the **hello-world-plugin** option.

### Configure Your Plugin

Maven will prompt you for several configuration values:

* artifactId: This is your plugin’s unique identifier. Choose a descriptive name without the words **“jenkins”** or **“plugin”.**
    

```plaintext
Example: release-gate, security-scanner, deployment-validator
```

* groupId: Follows Java package naming conventions.
    

```plaintext
Example: io.jenkins.plugins.yourcompany
```

* version: The initial version number (defaults to 1.0-SNAPSHOT).
    
* package: Your Java package structure.
    

```plaintext
Example: io.jenkins.plugins.releaseagte
```

## Troubleshooting Plugin Generation

If `archetype:generate` hangs or fails, try using an empty Maven settings file to bypass repository mirror issues:

```bash
echo '<settings/>' > /tmp/empty.xml
mvn -s /tmp/empty.xml -U archetype:generate -Dfilter=io.jenkins.archetypes:
```

## Understanding the Project Structure

After generation, your plugin directory will contain:

```plaintext
your-plugin/
├── pom.xml                 # Maven build configuration
├── src/
│   ├── main/
│   │   ├── java/          # Java source code
│   │   └── resources/     # Configuration files, Jelly views
│   └── test/
│       ├── java/          # Unit and integration tests
│       └── resources/
├── work/                  # Jenkins home directory (created on first run)
└── target/               # Compiled artifacts (created during build)
```

## Build Your Plugin

Navigate into your plugin directory and run:

```bash
mvn verify
```

This command:

* Compiles your code
    
* Runs unit tests
    
* Executes static analysis
    
* Packages the plugin as an `.hpi` file.
    

A successful build confirms that your plugin skeleton is properly configured.

---

# Building and Testing Locally

One of the most powerful features of Jenkins plugin development is the ability to run a local Jenkins instance with your plugin automatically loaded.

## Running Jenkins Locally

From your plugin directory, execute:

```bash
mvn hpi:run
```

You can specify a custom port if 8080 is already in use:

```bash
mvn hpi:run -Dport=5000
```

## Accessing Your Test Instance

Once the build completes, you’ll see:

```plaintext
INFO: Jenkins is fully up and running
```

Open your browser and navigate to:

```plaintext
http://localhost:8080/jenkins/ 
```

**Important Notes**:

* The Jenkins home directory is stored in `work/` within your plugin directory
    
* Configuration and build data persist across runs
    
* Changes to your plugin code require restarting the Maven process
    

## Testing Your Plugin

To test your plugin’s functionality:

1. Create a new **Freestyle project** in your local Jenkins instance
    
2. Add a build step that uses your custom plugin functionality
    
3. Configure the step with test parameters
    
4. Save the project and click **Build Now**
    
5. Review the **Console Output** to verify your plugin’s behavior
    

## Iterative Development

The typical development workflow looks like this:

1. Make code changes to your plugin
    
2. Stop the running `mvn hpi:run` process (Ctrl+C)
    
3. Run `mvn hpi:run` again to reload with your changes
    
4. Test the new functionality in your local Jenkins instance
    
5. Repeat until satisfied
    

---

# Hosting Your Plugin in the Jenkins Marketplace

Once you’ve developed and tested your plugin, you can host it in the Jenkins ecosystem, making it available to millions of Jenkins users worldwide.

The hosting process involves four distinct phases:

## Phase 1: Preparation

Before requesting hosting, ensure you’ve completed these prerequisite steps:

### Code and Conventions

* Finalize your plugin’s code
    
* Follow Jenkins naming conventions for:
    
    * Plugin name
        
    * GroupID
        
    * Commit messages
        
* Ensure code quality and adherence to best practices
    

### Public Repository

Your plugin’s source code must be available in a **public GitHub repository** under your personal account.

### License

Include a license file in your repository. The **MIT License** is strongly preferred by the Jenkins community.

```plaintext
Example: Add a LICENSE file with MIT license text
```

### GitHub Account

You need a GitHub account that will act as the maintainer for the plugin.

### Jenkins Community Account

Sign up for an account at [accounts.jenkins.io](https://accounts.jenkins.io/). This grants you access to:

* Jenkins Jira issue tracker
    
* Maven repository credentials (for releases)
    
* Community forums and mailing lists
    

### Documentation

Prepare comprehensive documentation including:

* Installation instructions
    
* Configuration guide
    
* Usage examples
    
* Troubleshooting tips
    

---

## Phase 2: Requesting Plugin Hosting

With preparation complete, you can now request official hosting.

### Open a Hosting Request

1. Log in to GitHub
    
2. Navigate to the repository-permissions-updater repository
    
3. Create a new issue using the hosting request template
    
4. Fill out all fields completely:
    
    1. Plugin name
        
    2. Repository URL
        
    3. GitHub username
        
    4. Purpose and description
        
    5. License information
        

### Review Process

The Jenkins Hosting team will review your request within several days. They may:

* Request changes to your code or documentation
    
* Ask for clarification on plugin functionality
    
* Suggest improvements to naming or structure
    

**Be responsive to feedback**—implementing requested changes promptly helps expedite the approval process.

### Repository Fork and Transfer

Once approved, the automated process will:

1. Fork your repository into the `jenkinsci` GitHub organization
    
2. Invite you to join the `jenkinsci` organization with **admin access** to your plugin repository
    
3. Request that you **delete your original repository** or **private your original repository for internal purposes and let the forked repository be the only source of truth public-ally available.**
    

**Why delete your original repository?**

* Ensures the `jenkinsci` repository becomes the canonical source
    
* Improves discoverability in code searches
    
* Encourages community contributions to the correct location
    
* Prevents confusion about which repository is official.
    

You can recreate your personal repository later by forking from the `jenkinsci` version if needed for experimental work.

---

## Phase 3: Post-Approval Setup

With your plugin now part of the `jenkinsci` organization, configure it for builds, releases, and integration.

### Enable CI Builds

Create a `Jenkinsfile` in the root of your plugin repository to enable Continuous Integration:

```plaintext
buildPlugin(
  // Use Java 21 for building
  configurations: [
    [platform: 'linux', jdk: 21],
    [platform: 'windows', jdk: 21]
  ]
)
```

This allows the Jenkins project’s CI/CD infrastructure to:

* Automatically build your plugin on every commit
    
* Run tests across multiple platforms
    
* Verify compatibility with different Jenkins versions
    

### Request Upload Permissions

To release your plugin to the Jenkins Maven repository, you need upload permissions.

Two options are available:

**Option 1: Manual Releases** A pull request for upload permissions may be opened automatically. If not, create one yourself by:

1. Forking the [repository-permissions-updater](https://github.com/jenkins-infra/repository-permissions-updater) repository
    
2. Adding your plugin’s metadata
    
3. Submitting a pull request
    

**Option 2: Automated Releases (Recommended)** Configure your plugin for automatic releases on every push to the main branch. This approach:

* Requires only GitHub write permissions
    
* Eliminates the need for local Maven credentials
    
* Simplifies the release process
    
* Reduces manual overhead
    

### Categorize Your Plugin

Help users discover your plugin by adding appropriate categories and labels. Update your `pom.xml` with category metadata:

```xml
<properties>
  <jenkins.version>2.387.3</jenkins.version>
  <plugin.category>misc</plugin.category>
</properties>
```

Available categories include:

* build: Build tools and steps
    
* scm: Source control management
    
* security: Security and compliance
    
* notification: Notifications and messaging
    
* misc: Miscellaneous
    

Add labels in your plugin documentation to improve searchability in the Plugin Manager and on [plugins.jenkins.io](http://plugins.jenkins.io).

### Integrate with Jira (Optional)

If you use Jira for issue tracking, link commits directly to issues:

1. Go to your GitHub repository settings
    
2. Navigate to **Autolink references**
    
3. Add the following rule:
    
    1. Reference prefix: JENKINS-
        
    2. Target URL: [https://issues.jenkins.io/browse/JENKINS-&lt;num&gt;](https://issues.jenkins.io/browse/JENKINS)
        

Now, commit messages like `Fix JENKINS-12345` will automatically link to the corresponding Jira issue.

---

## Phase 4: Community Engagement

As a plugin maintainer, engaging with the Jenkins community enhances your plugin’s success and your own development skills.

### Join the Mailing List

Subscribe to the [jenkinsci-dev mailing list](https://groups.google.com/g/jenkinsci-dev) to:

* Stay informed about Jenkins development news
    
* Participate in architectural discussions
    
* Get help with plugin development questions
    
* Announce new releases and features
    

### Communicate Openly

All communication regarding your plugin should happen in **public channels**:

* Mailing lists for general discussions
    
* Jira for bug reports and feature requests
    
* GitHub issues and pull requests for code-related topics
    

Avoid private messages or direct emails. Public communication:

* Creates a searchable knowledge base
    
* Allows others to learn from discussions
    
* Fosters transparency and trust
    
* Encourages community participation
    

### Respond to Issues and Pull Requests

As a maintainer, you’re responsible for:

* Responding to bug reports and feature requests
    
* Reviewing and merging community pull requests
    
* Maintaining backward compatibility when possible
    
* Communicating deprecation and breaking changes
    

### Version and Release Your Plugin

When releasing new versions:

1. Update version in `pom.xml`
    
2. Document changes in a `CHANGELOG` or release notes
    
3. Follow semantic versioning: `MAJOR.MINOR.PATCH`
    
    1. MAJOR: Breaking changes
        
    2. MINOR: New features, backward compatible
        
    3. PATCH: Bug fixes
        

Use Maven to create a release:

```bash
mvn release:prepare release:perform
```

Or, if using automated releases, simply tag your commit:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

---

# Best Practices for Plugin Development

### Security Considerations

Security is paramount in Jenkins plugins. Follow these guidelines:

* **Validate all user inputs** to prevent injection attacks
    
* **Escape output** in views to prevent XSS vulnerabilities
    
* **Use Jenkins’ Permission APIs** for access control
    
* **Avoid storing sensitive data** in plain text
    
* **Follow the** **OWASP Top 10** guidelines
    

### Code Quality

Maintain high code quality standards:

* Write comprehensive **unit tests** (aim for &gt;80% coverage)
    
* Include **integration tests** using `JenkinsRule`
    
* Use **static analysis tools** like SpotBugs and Checkstyle
    
* Follow **Java coding conventions**
    
* Document your code with clear Javadoc comments
    

### Backward Compatibility

Jenkins has a long support lifecycle. When developing plugins:

* Choose an appropriate **Jenkins baseline version** (the minimum Jenkins version your plugin supports)
    
* Avoid breaking changes when possible
    
* Deprecate features before removing them
    
* Test against multiple Jenkins versions
    

### Documentation

Good documentation is essential:

* Provide a comprehensive **README** in your repository
    
* Include **usage examples** for common scenarios
    
* Document **configuration** **options** clearly
    
* Maintain a **CHANGELOG** of version updates
    
* Consider creating **video tutorials** or **blog posts**
    

## **Example Use Cases**

To inspire your plugin development journey, here are some real-world use cases:

### **Security Gate Plugin**

Integrate security scanning tools directly into the CI/CD pipeline, allowing builds to be blocked or marked unstable based on vulnerability scans or policy violations.

### **Custom Deployment Plugin**

Implement deployment steps specific to your organization's infrastructure, supporting multiple environments and deployment strategies.

### **Job Discovery and Monitoring**

Periodically scan Jenkins for all jobs and send their metadata to external monitoring platforms for analysis and compliance tracking.

### **Integration with Proprietary Tools**

Connect Jenkins with internal tools like ticketing systems, deployment platforms, or custom databases that don't have existing integrations.

### **Policy Enforcement**

Enforce organizational policies such as mandatory code reviews, required test coverage thresholds, or approval workflows before production deployments.

---

# **Common Challenges and Solutions**

### **Challenge: Long Build Times**

**Solution**: Use the Maven HPI Plugin's incremental build features and optimize your test suite.

### **Challenge: Debugging Plugin Issues**

**Solution**: Use \``mvn hpi:run -Dorg.slf4j.simpleLogger.defaultLogLevel=debug` for detailed logging.

### **Challenge: Dependency Conflicts**

**Solution**: Carefully manage plugin dependencies and use the `provided` scope for Jenkins core dependencies.

### **Challenge: UI Not Updating**

**Solution**: Clear your browser cache or use incognito mode when testing UI changes.

### **Challenge: Plugin Not Loading**

**Solution**: Check the Jenkins system log for errors and verify your plugin's dependencies are satisfied.

---

# **Resources and Further Learning**

### **Official Documentation**

* [Jenkins Plugin Tutorial](https://www.jenkins.io/doc/developer/tutorial/)
    
* [Plugin Development Guide](https://www.jenkins.io/doc/developer/plugin-development/)
    
* [Extension Points Reference](https://www.jenkins.io/doc/developer/extensions/)
    
* [Maven HPI Plugin Documentation](https://jenkinsci.github.io/maven-hpi-plugin/)
    

### **Community Resources**

* [jenkinsci-dev Mailing List](https://groups.google.com/g/jenkinsci-dev)
    
* [Jenkins Community Chat](https://www.jenkins.io/chat/)
    
* [Jenkins Jira](https://issues.jenkins.io/)
    
* [Stack Overflow - Jenkins Tag](https://stackoverflow.com/questions/tagged/jenkins)
    

### **Code Examples**

* Browse existing plugins for examples: [JenkinsCI GitHub Organization](https://github.com/jenkinsci)
    

---

# **Conclusion**

Creating a Jenkins plugin opens up endless possibilities for extending and customizing your CI/CD pipelines. Whether you're solving a unique problem for your organization or contributing to the broader Jenkins community, plugin development is a rewarding journey that combines software engineering with real-world automation challenges.

The path from local development to publishing in the Jenkins Marketplace may seem daunting at first, but by following this guide, you'll have a clear roadmap to success. Remember:

* **Start small**: Begin with simple functionality and iterate
    
* **Test thoroughly**: Use local Jenkins instances to validate behavior
    
* **Engage with the community**: The Jenkins community is welcoming and helpful
    
* **Document well**: Good documentation benefits both users and future maintainers
    
* **Maintain actively**: Respond to issues and keep your plugin updated
    

Your plugin could become an essential tool for thousands of Jenkins users worldwide. The Jenkins ecosystem thrives on community contributions, and every plugin—no matter how specialized—adds value to the platform.

Ready to get started? Set up your environment, generate your first plugin, and join the vibrant community of Jenkins plugin developers today!

---

**About the Author**: This guide is based on hands-on experience developing and hosting Jenkins plugins, following official Jenkins documentation and community best practices.

**Questions or Feedback?** Reach out to the Jenkins community through the mailing lists or chat channels listed in the Resources section.
# Fletchersan.github.io

A Jekyll-based GitHub Pages site with blog posts about programming challenges, system design, and technical topics.

## 🚀 Quick Start

### Prerequisites

- **Ruby**: Version 3.4+ (recommended: use Homebrew Ruby instead of system Ruby)
- **Homebrew**: For installing Ruby and managing dependencies
- **Git**: For version control

### Installation

1. **Install Homebrew Ruby** (recommended over system Ruby):
   ```bash
   brew install ruby
   ```

2. **Add Ruby to your PATH**:
   ```bash
   echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
   source ~/.zshrc
   ```

3. **Verify Ruby version**:
   ```bash
   ruby --version
   # Should show Ruby 3.4+ (not 2.6.x)
   ```

4. **Install Bundler**:
   ```bash
   gem install bundler:2.4.14
   ```

5. **Install dependencies**:
   ```bash
   bundle install
   ```

## 🛠️ Development

### Start Development Server

```bash
bundle exec jekyll serve --livereload
```

**What this does:**
- Builds your Jekyll site
- Starts a local server at `http://localhost:4000`
- Enables live reloading (browser auto-refreshes on file changes)
- Watches for file changes and rebuilds automatically

### Alternative Commands

```bash
# Basic serve (no auto-refresh)
bundle exec jekyll serve

# Custom port
bundle exec jekyll serve --port 4001 --livereload

# Include draft posts
bundle exec jekyll serve --drafts --livereload

# Incremental build (faster rebuilds)
bundle exec jekyll serve --incremental --livereload
```

### Build Site

```bash
# Build to docs/ directory
bundle exec jekyll build ./docs

# Build to custom directory
bundle exec jekyll build --destination ./_site
```

## 📁 Project Structure

```
Fletchersan.github.io/
├── _posts/                 # Blog posts (Markdown)
├── _layout/               # Jekyll layouts
├── _includes/             # Reusable HTML components
├── _sass/                 # SCSS stylesheets
├── assets/                # Images, CSS, JS files
├── docs/                  # Built site (generated)
├── Gemfile                # Ruby dependencies
├── _config.yml            # Jekyll configuration
└── README.md              # This file
```

## 🔧 Configuration

### Jekyll Configuration (`_config.yml`)
- Site title, description, and metadata
- Plugin settings
- Build destination and source directories

### Gemfile
- Jekyll version and plugins
- Ruby gem dependencies
- Compatibility fixes for Ruby 3.4+

## 🐛 Troubleshooting

### Common Issues

#### 1. Ruby Version Problems
**Problem**: `Could not find 'bundler'` or permission errors
**Solution**: Use Homebrew Ruby instead of system Ruby
```bash
brew install ruby
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

#### 2. Missing CSV/Base64 Libraries
**Problem**: `cannot load such file -- csv` or `cannot load such file -- base64`
**Solution**: These gems are already included in the Gemfile for Ruby 3.4+ compatibility

#### 3. Build Failures
**Problem**: Jekyll build fails with dependency errors
**Solution**: Clean install
```bash
rm -rf vendor/
bundle install
```

#### 4. Port Already in Use
**Problem**: `Address already in use` error
**Solution**: Use a different port
```bash
bundle exec jekyll serve --port 4001 --livereload
```

### Sass Deprecation Warnings

The build will show Sass deprecation warnings (these are harmless):
- `@import` rules are deprecated → Use `@use` instead
- `lighten()`/`darken()` functions are deprecated → Use `color.adjust()` instead
- Division operators are deprecated → Use `math.div()` or `calc()` instead

These warnings don't affect functionality and can be ignored for now.

## 📝 Adding Content

### New Blog Post
1. Create a new Markdown file in `_posts/`
2. Use the format: `YYYY-MM-DD-Title.md`
3. Add front matter at the top:
   ```markdown
   ---
   layout: post
   title: "Your Post Title"
   date: 2025-08-23
   categories: [category1, category2]
   ---
   ```

### Adding Images
1. Place images in `assets/` directory
2. Reference them in markdown:
   ```markdown
   ![Alt Text]({{ site.baseurl }}/assets/image-name.png)
   ```

## 🚀 Deployment

### GitHub Pages
This site is configured for GitHub Pages deployment:
- Source: `docs/` directory
- Build command: `bundle exec jekyll build ./docs`
- Auto-deploy on push to main branch

### Manual Deployment
```bash
# Build the site
bundle exec jekyll build ./docs

# Commit and push changes
git add .
git commit -m "Update site"
git push origin main
```

## 🔍 Useful Commands

```bash
# Check Ruby version
ruby --version

# Check Bundler version
bundle --version

# List installed gems
bundle list

# Update gems
bundle update

# Clean build artifacts
rm -rf docs/ _site/

# Check Jekyll version
bundle exec jekyll --version
```

## 📚 Resources

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [GitHub Pages](https://pages.github.com/)
- [Minima Theme](https://github.com/jekyll/minima)
- [Ruby Installation](https://www.ruby-lang.org/en/documentation/installation/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test locally with `bundle exec jekyll serve --livereload`
5. Submit a pull request

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

**Note**: This README is automatically generated and updated. For the most current information, check the repository or run `bundle exec jekyll serve --help`.

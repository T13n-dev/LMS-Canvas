# Issue 1: Config languages for canvas LMS
1. Create file vi.yml root/config/locales
2. Simple configuration for locales.yml
```bash
vi:
  bigeasy_locale: vi
  locales:
    vi: Tiếng kinh (VN)
```
3. Set default lang application.rb
```bash
    config.i18n.load_path << Rails.root.join('config', 'locales', 'locales.yml')
    config.i18n.default_locale = :vi
```
# Issue 2: Rail run and re-install assets
```bash 
bundle exec rails server
bundle exec rake canvas:compile_assets
```

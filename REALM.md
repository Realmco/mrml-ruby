# Realm's build of mrml-ruby

Realm renders its emails with MRML (realm-app RLM-7409). Upstream merged mrml 6.0.1 with `mj-style inline`
support (#38) but hasn't released a gem since 1.10.0, which still has mrml 5.1. This fork's `realm` branch
builds precompiled gems from upstream's `840f48d`, and Realm installs them from the public `realm-gems` S3
bucket:

```ruby
source "https://realm-gems.s3.amazonaws.com" do
  gem "mrml", "1.11.0.pre.840f48d"
end
```

Go back to the rubygems.org gem once upstream releases one with mrml 6 (hardpixel/mrml-ruby#40), and
retire this fork and the bucket.

## Rebuilding

Needed for a new Ruby minor version (the gems hold one binary per Ruby: 3.4 and 4.0 now), a new platform,
or a newer upstream commit.

1. On the `realm` branch, change what's needed: `ruby-versions` or the platform list in
   `.github/workflows/cross-compile.yml`, or merge upstream. Give it a new version in
   `lib/mrml/version.rb`, `1.11.0.pre.<upstream short sha>` with a `.1`, `.2` suffix for a rebuild of the
   same commit. Published files never change in place.
2. Push, then push the tag `v<version>`. The Cross Compile workflow builds every gem into its `cross-gem`
   artifact.
3. Publish, from a folder holding the bucket's current files under `gems/` plus the new ones:

   ```sh
   aws s3 sync s3://realm-gems/gems gems --profile realm
   gh run download <run id> --repo Realmco/mrml-ruby --name cross-gem --dir new && cp new/*.gem gems/
   gem install rubygems-generate_index   # once
   gem generate_index --directory . --modern --compact
   aws s3 sync . s3://realm-gems --profile realm --exclude 'new/*'
   ```

4. Bump the version in realm-app's Gemfile and run `bundle install`.

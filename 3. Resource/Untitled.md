https://github.com/sproogen/modern-resume-theme?tab=readme-ov-file


1. zip 파일 다운 받아서 압축 풀기 
2. `git init`
3. [jekyll 설치](https://jekyllrb.com/docs/installation/ubuntu/)

Ubuntu
```shell
sudo apt-get install ruby-full build-essential zlib1g-dev

echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc 
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc 
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc 
source ~/.bashrc

gem install jekyll bundler
```


디렉터리 이동 후 실행시 에러 발생
```shell
bundle install

modern-resume-theme-2.0.10 requires ruby version ~> 2.0, which is incompatible
with the current version, ruby 3.2.3p157
```

- ruby 버전 3.x대를 지원하지 않음
- 2.x대로 다운그레이드 필요 


**✅ 1. rbenv & ruby-build 설치** (Chat-GPT)

```shell
sudo apt update
sudo apt install -y git curl build-essential libssl-dev libreadline-dev zlib1g-dev

# rbenv 설치
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init - bash)"' >> ~/.bashrc
source ~/.bashrc

# ruby-build 설치
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```


```shell
rbenv install 2.7.8
/usr/bin/env: `bash\r': 그런 파일이나 디렉터리가 없습니다` /usr/bin/env: #! 줄에서 옵션을 전달하려면 -[v]S 옵션을 사용하십시오
```
- 개행 문자 처리가 문제가 됨 

```shell
sudo apt install dos2unix

readlink -f ~/.rbenv/bin/rbenv

dos2unix /home/leejinwoo/.rbenv/libexec/rbenv

// 요거 하고 나서야 인식이 됨
find ~/.rbenv -type f -exec dos2unix {} \;
```


다시 저장소로 이동 후 시키는데로 따라한다
```shell
bundle install



```
- nokogiri와 bundler 등 버전 이슈 발생

```
rbenv install 2.6.6
rbenv global 2.6.6

gen uninstall nokogiri
gem install nokogiri -v 1.10.8 --platform=ruby

gem install bundler:2.1.4
```

```
bundle exec jekyll serve
```

http://localhost:4000 으로 접속하면 로컬에서 확인 가능한듯 하다 

---

정적 웹 호스팅

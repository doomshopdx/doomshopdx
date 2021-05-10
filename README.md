#!/usr/bin/env python3
# Usage:
#    pip3 install PyGithub
#
#    echo someusername | python3 check_github_usernames.py
#    # or
#    python3 check_github_usernames.py < list_of_usernames.txt
#
#    [+] Starting to check 1212 usernames on GitHub...
#    X agq (is taken)
#    X ahq (is taken)
#    X aqf (is taken)
#    X aqg (is taken)
#    X aqp (is taken)
#    ...
#    √ q3z (is available)
#    ...

# FYI Github also has a anti-username-squatting policy, you can request a taken but inactive username be transferred to you (with some restrictions)
# https://docs.github.com/en/github/site-policy/github-username-policy#name-squatting-policy

import sys
import argparse

from time import sleep
from datetime import datetime
from github import Github, UnknownObjectException

GITHUB_USERNAME = ''    # fill in a valid github username and password to use to access the API
GITHUB_PASSWOWRD = ''
# or
GITHUB_API_TOKEN = ''   # fill in a GitHub personal access token (PAT) to use to access the API
# https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token


def check_ratelimit(github):
    limits = github.get_rate_limit()
    if limits.core.remaining <= 2:
        seconds = int((limits.core.reset - datetime.utcnow()).total_seconds()) + 2
        if seconds > 300:
            sys.stderr.write(f'[!] Rate limited, waiting {seconds}s until {limits.core.reset} (UTC)...\n')
        sleep(seconds)

def save_user(c, user):
    print(user.id, user.login, user.email, user.name, user.location)


def check_usernames(github, usernames: list):
    sys.stderr.write(f'[+] Starting to check {len(usernames)} usernames on GitHub...\n')

    try:
        check_ratelimit(github)
        for username in usernames:
            try:
                assert github.get_user(username).id
                print(f'X {username} (is taken)')
            except (UnknownObjectException, AssertionError):
                print(f'√ {username} (is available)')

            check_ratelimit(github)

    except (KeyboardInterrupt, SystemExit) as e:
        sys.stderr.write(f'\n[X] Stopped ({e.__class__.__name__})\n')


if __name__ == '__main__':
    assert GITHUB_API_TOKEN or (GITHUB_USERNAME and GITHUB_PASSWOWRD), (
        'You must set a GITHUB_API_TOKEN or a GITHUB_USERNAME and GITHUB_PASSWOWRD at the top of the file')
    github = Github(GITHUB_API_TOKEN) if GITHUB_API_TOKEN else Github(GITHUB_USERNAME, GITHUB_PASSWOWRD)

    usernames = [
        username.strip()
        for username_row in sys.stdin.readlines()
            for username_chunk in username_row.split(',')
                for username in username_chunk.split(' ')
        if username.strip()
    ]

    check_usernames(github=github, usernames=usernames)
   

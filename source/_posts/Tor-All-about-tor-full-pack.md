---
title: (Tor)-All-about-tor-full-pack
date: 2018-07-16 12:06:04
categories:
    - ToolKit
tags: 
    - Tor
---

# All About Tor

Version 2019 - 02

# Agenda 

 * Fill in this section
 * With your agenda for the day
 * To help your audience stay focused!

# Let's begin

 * Do you use Tor?
 * If not, why not?
 * If yes, do you have any questions, concerns, issues, or doubts about using it?
 * Do you teach others about Tor?

# What is Tor?

 * Free software and an open network
 * Mitigates against tracking, surveillance and censorship
 * Run by a US non-profit and volunteers from all over the world
 * It's Tor, not TOR

# Why do we need Tor?

 * Government mass and targeted surveillance
 * The business model of the Internet: big data, advertising, non-consensual tracking
 * Surveillance threats from family, bosses, bad people on the Internet

# Why do you need Tor?

 * Let’s discuss the work you do 
 * Adversaries and challenges 
 * Mitigations
 * How Tor can help

# little-t Tor or core tor

 * Tor the network daemon (a computer program)
 * Presents a SOCKS or http proxy
 * Location and source anonymity, similar to a VPN or regular proxy (but better!)
 * Network of relays in many parts of the world

# Who can see your activity without Tor or HTTPS?


# Who can see your activity with Tor and HTTPS?

# Tor Browser

 * little-t tor plus patched Firefox
 * Anyone snooping can’t see the websites you visit
 * Websites can’t track you or see other sites you visit (cross-tracking)
 * Prevents other privacy violations like fingerprinting or 3rd party cookies
 * Writes nearly nothing to disk
 * No browser history
 * Cross platform: Windows, macOS, Linux and Android

# Tor Browser in other languages

 * Go to https://torproject.org
 * Select the language  on dropdown menu 
 * Or select “Download in another language or platform”

# Downloading Tor Browser

 * The safest way to download is from https://torproject.org

 * If https://torproject.org is blocked, try mirrors
  * http://tor.calyxinstitute.org/
  * https://tor.eff.org/

 * Or try GetTor - email gettor@torproject.org and in the message write “windows”, “osx” or “linux” (no quotes, no subject line)

# Running Tor Browser the first time

# Using Tor Browser

 * Default search engine: DuckDuckGo
 * Bundled with NoScript, HTTPS Everywhere
 * You should not add any other extensions nor enable any plugins (eg Flash)!
   * Best practices:
   Websites won’t know anything about you unless you login and tell them 

# Clicking on the site information menu will show your current circuit (and “New Circuit for this site” option)

# Updating Tor Browser

# Uninstalling Tor Browser

Uninstalling Tor Browser is as easy as moving the folder to the trash! Then, empty the trash.

Default Tor Browser folder locations:

 * Windows: desktop
 * macOS: move the Tor Browser application to Trash and also the TorBrowser-Data folder  (~/Library/Application Support/)
 * Linux: home, or look for a name like “tor-browser_en-US” 

# Troubleshooting Tor Browser

 * Is your system clock correct?
 * Is the browser already running?
 * Are you being censored?
 * Is your antivirus or firewall blocking Tor?
 * Do you have a very old operating system?
 * Try uninstalling and reinstalling
 * Get help at https://support.torproject.org 

# More Tor Browser

# The “onion” menu

 * New Identity: completely refreshes Tor Browser and its circuits; wipes all history; closes all tabs
 * Tor network settings: censorship mitigation options; access to Tor log
 * Check for Tor Browser update 

# Security Slider

# NoScript

 * It’s not advisable to change settings in the “options” menu
 * For example, adding sites to the “whitelist” can result in fingerprinting
 * Instead, only “temporarily trust” blocked objects 

# DuckDuckGo

 * DuckDuckGo is the default search engine in Tor Browser
 * Using Tor Browser prevents DDG from tracking users, even if they wanted to (they claim not to)
 * Duckduckgo.com or https://3g2upl4pq6kufc4m.onion/

# Plugins, add-ons, Javascript

 * Do not add any new add-ons/extensions to Tor, and don’t enable any plugins
 * For example, Flash plugin can reveal your real location 
 * Javascript is enabled by default, but is sanitized to preserve anonymity
 * To prevent possible Javascript vulnerabilities, use the “safest” setting in the security slider

# Mobile Tor

# Things to know about mobile Tor

 * The design of mobile devices makes full privacy impossible
 * Mobile Tor is best for censorship prevention
 * Can also provide better privacy for some threat models
 * We’re making it better all the time and better options for mobile devices are coming out soon

# Tor Browser for Android

 * You don’t need to install two APPs (Orbot and Orfox) anymore
 * Find it in the Play Store
 * Or Guardian Project repository in F-Droid
 * Or download .apk 

# Onion Browser

 * Tor Browser for iOS
 * Find it in the App Store
 * Lots of fake Tor Browsers for iOS
 * Very rudimentary
 * Crashes on sleep

# Using Orbot

 * Tor proxy for Android
 * Find it on the Play Store or F-Droid
 * Use it to run other Apps through Tor (like Twitter)
 * Click start to run 
 * You can choose your exit country if you want (some countries don’t have exits!)

# Using Orbot

 * Toggle “VPN mode” on main screen
 * Then click “Orbot-enabled apps”
 * Then select the apps you want to proxy with Tor

# How do I get help using Tor?

# Help using Tor

 * Tor Browser Manual: tb-manual.torproject.org
 * Support Portal: https://support.torproject.org
 * Open mailing lists: https://lists.torproject.org 
 * Chat: IRC OFTC network - easy access through https://webchat.oftc.net channel #tor
 * https://tor.stackexchange.com

# If you find a bug in Tor
 
 * https://trac.torproject.org
 * Create a login
 * Search for your issue to find any existing tickets
 * If no ticket opened, open a new ticket with detailed description of the problem

# Circumventing censorship with Tor

# What do you do when  Tor is blocked?

# I downloaded Tor Browser, but it won’t connect

 * If this screen takes a long time and does not connect, you may need a bridge or pluggable transport

# When torproject.org is blocked

 * Mirrors
  * https://tor.eff.org/ 
  * http://tor.calyxinstitute.org/ (if https is blocked)
 * GetTor email: gettor@torproject.org
  * Contact from a Gmail or Riseup account
 * Flash drive with Tor on it from someone you trust
  * Get the EXE, DMG, tar.xz, don’t copy the installed folder
 * Downloading Tor from a random website is dangerous!

# Bridges and pluggable transports

 * Bridges are relays that are not listed publicly
 * Get bridges directly from Tor Browser (moat)
 * Or from the website https://bridges.torproject.org or send an email to bridges@torproject.org from a Gmail, or Riseup.net account
 * Or get a bridge address from a trusted person
 * Pluggable transports can be used like bridges to disguise Tor traffic (also called “built-in bridges”)

# Bridges and pluggable transports

# Request a bridge

 * moat method

# Or select a built-in bridge

# Pluggable transports

 * obfs4: makes Tor traffic look random; works in many situations including China (if not, try meek)
 * fte: makes Tor traffic look like regular HTTP traffic
 * meek-azure: makes it look like Microsoft traffic; works in China
 * snowflake: proxies traffic through temporary proxies using WebRTC (under development) https://snowflake.torproject.org

# OONI

 * Open Observatory of Network Interference:https://ooni.torproject.org
 * Country-level reports of specific censorship tools in use on certain websites
 * View their reports:https://explorer.ooni.torproject.org
 * Or use your own OONI Probe to test websites: available in App Store and PlayStore

# Sharing content anonymously with Tor 

# What are Onion Services?

The regular internet allows adversaries to see what you are sharing and with whom, whether you're using Dropbox etc, downloading it from email or through your browser...

...so Tor devised a sneaky way to hide both the file data and the related metadata!

# Onion Services

 * Protection for both the user and the server
 * User learns about xyz.onion
 * Client and service meet at rendezvous point in the Tor cloud
 * End-to-end encrypted without HTTPS
 * Connections never go out to the “vanilla” internet

# OnionShare

 * Secure, private, anonymous file sharing done easy, built on top of the Tor network
 * Uses onion services to securely send files
 * Creates an onion service where the file can be downloaded
 * No need to trust third parties like Dropbox
 * Download from https://onionshare.org

# Connection

 * OnionShare connects to Tor network
 * Click “add” then find the file you want to share.

# Sharing

Once the file is added, click “start sharing”

# Sharing

 * Copy the xyz.onion link and send it to your contact
 * Contact installs Tor Browser
 * When they finish downloading you’ll see this →

# Help Link

Ip hidden and TOR network configured: Visit https://check.torproject.org, you should see a message like: "Congratulations. This Browser is configured to use Tor."

Checking DNS Leaks: Visit https://dnsleaktest.com and make a extended test to see what are your DNS.


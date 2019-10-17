---
layout: post
title: Secure your Web App in Azure at DDD14
date: '2019-10-16'
author: Frans Lytzen
tags: Azure SQL Security Web
modified_time: '2019-10-16'
excerpt: I was recently asked whether a SHA256 hash could have a "/" in it. If you know how hashes work, you probably know that the question doesn't make sense. But I thought it was a good reason to write a bit about how hashes work and, specifically, how they manifest in our coding. Personally, I've been doing this for a long time and some of the details were rather opaque to me until recently.
---
I had the great pleasure of giving an updated version of my ["Secure your Web App in Azure" talk](/talks/Securing_web_apps_in_azure.html) talk at [Developer Developer Developer 14](https://developerdeveloperdeveloper.com/) in Reading on 12 October 2019.

A video of the whole talk is available below.

I touch on a whole range of Azure technologies, but mostly I introduce and expand on a simple framework to think about and manage your exposure. 



## Example of exposure and mitigation
<table>
<tr>
    <th></th>
    <th>External Actors</th>
    <th>Internal Actors</th>
</tr>
<tr>
    <th>PREVENT</th>
    <td>
        <ul>
            <li>
                Secure your code – see Troy Hunt’s courses as a starting point.
            </li>
            <li>
                Lock down your servers
            </li>
            <li>
                Use Firewalls and Intrusion Detection/Prevention Systems
            </li>
            <li>
                Encrypt everything in transit
            </li>
        </ul>
    </td>
    <td>
        <ul>
            <li>
                Protect your passwords/secrets
            </li>
            <li>
                Process for granting and removing access
            </li>
            <li>
                Use Azure AD for all access, including SQL
            </li>
            <li>
                Audit who has access on a regular basis and remove unnecessary access
            </li>
        </ul>
    </td>
</tr>

<tr>
    <th>DETECT</th>
    <td>
        <ul>
            <li>
                Log and alert on any unusual application activity
                <ul>
                    <li>
                        403s and 404s
                    </li>
                    <li>
                        Failed logins
                    </li>
                    <li>
                        High CPU/memory, increased load
                    </li>
                    <li>
                        Etc
                    </li>
                </ul>
            </li>
            <li>
                Use Advanced Threat Protection
            </li>
        </ul>
    </td>
    <td>
        <ul>
            <li>
                Log and alert on all access to the backend by internal users
            </li>
            <li>
                Log and alert on unusual access patterns by application users
            </li>
            <li>
                Consider DLP tools
            </li>
        </ul>
    </td>
</tr>

<tr>
    <th>MITIGATE</th>
    <td colspan="2">
        <ul>
            <li>
                Encrypt sensitive data at the application layer
            </li>
            <li>
                Have ways of locking out certain users or IP addresses
            </li>
            <li>
                For very sensitive systems, consider multi-layered architectures to contain breaches
            </li>
        </ul>
    </td>
    
</tr>
</table>
<br/>

## Video of the whole talk
<iframe width="560" height="315" src="https://www.youtube.com/embed/HZgjlTi7OiA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
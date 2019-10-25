---
layout: post
title: Secure your Web App in Azure at DDD14
date: '2019-10-16'
author: Frans Lytzen
tags: Azure SQL Security Web
modified_time: '2019-10-16'
excerpt: An overview of how to secure Azure Web Apps, touching on virtual networks, encryption, key vault and monitoring.
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

<br />
## Slides
View the slides on Slide Share:
<iframe src="//www.slideshare.net/slideshow/embed_code/key/DbJSjU5lVqZNUN" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/FransLytzen/secure-your-azure-web-app-2019" title="Secure your Azure Web App 2019" target="_blank">Secure your Azure Web App 2019</a> </strong> from <strong><a href="https://www.slideshare.net/FransLytzen" target="_blank">Frans Lytzen</a></strong> </div>

... or download from [GitHub](https://github.com/flytzen/SecurityTalk/blob/master/Secure%20Your%20Web%20App%20Presentation.pptx?raw=true)
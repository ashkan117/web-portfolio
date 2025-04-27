---
title: "Global Modal"
date: 2025-03-07T17:40:11Z
draft: true
---

In your layout define
<%= live_render(@socket, LiveBeatsWeb.PlayerLive, id: "player", session: %{}, sticky: true) %>

Sticky means
:sticky - an optional flag to maintain the LiveView across live redirects, even if it is nested within another LiveView. Note that this only works for LiveViews that are in the same live_session. If you are rendering the sticky view within make sure that the sticky view itself does not use the same layout. You can do so by returning {:ok, socket, layout: false} from mount.

I want to have a modal todo form
Source of truth, should it be the modal form or not
I do want the page to update afterwards

I'm assuming

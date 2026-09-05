import asyncio
import re
from telethon import TelegramClient, events
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger(__name__)

# ===== CONFIGURATION =====
API_ID = 39059969  # Your API ID
API_HASH = 'a62d1c49b11fbbc81d1e00a3e5170c54'
PHONE_NUMBER = '601124226877'

# Groups & Bots
WAIFU_BOT = '@HusbandosWaifusCheatsBot'
CHARACTER_BOT = '@Character_Catcher_Bot'

# Storage
processed_messages = set()
last_image_info = {}
saved_groups = []

client = TelegramClient('userbot', API_ID, API_HASH)

# ===== Get group link =====
async def get_group_link(chat_id):
    try:
        entity = await client.get_entity(chat_id)
        try:
            link = await client.export_invite_link(entity)
            return link
        except:
            if hasattr(entity, 'username') and entity.username:
                return f"https://t.me/{entity.username}"
            else:
                return f"Group ID: {chat_id}"
    except:
        return f"Chat ID: {chat_id}"

# ===== PART 1: Detect images =====
@client.on(events.NewMessage)
async def detect_image(event):
    global last_image_info

    try:
        if not event.is_group:
            return
        if event.message.id in processed_messages:
            return
        if event.message.forward:
            return
        if not event.message.photo:
            return

        # Check if from Character_Catcher_Bot
        if event.message.sender_id:
            try:
                sender = await event.message.get_sender()
                if sender and hasattr(sender, 'username'):
                    if sender.username != 'Character_Catcher_Bot':
                        return
            except:
                if event.message.text and 'Character_Catcher_Bot' not in event.message.text:
                    return

        processed_messages.add(event.message.id)

        chat_id = event.chat.id
        chat_title = event.chat.title or "Unknown Group"
        last_image_info = {
            'chat_id': chat_id,
            'chat_title': chat_title,
            'message_id': event.message.id
        }

        logger.info(f"✅ Image detected in: {chat_title}")

        # Save group link
        group_link = await get_group_link(chat_id)
        saved_groups.append({
            'chat_id': chat_id,
            'chat_title': chat_title,
            'link': group_link
        })

        # Send to Saved Messages
        await client.send_message(
            'me',
            f"📌 Card Detected!\n\n🏷️ Group: {chat_title}\n🔗 Link: {group_link}"
        )

        # Send image to Waifu Bot
        image_path = await event.message.download_media()
        await client.send_file(WAIFU_BOT, image_path)
        import os
        os.remove(image_path)

    except Exception as e:
        logger.error(f"❌ Error: {e}")

# ===== PART 2: Copy name and send ONLY /catch =====
@client.on(events.NewMessage(chats=[WAIFU_BOT]))
async def copy_name_and_catch(event):
    global last_image_info

    try:
        if not event.message.text:
            return
        if event.message.id in processed_messages:
            return
        processed_messages.add(event.message.id)

        message_text = event.message.text

        # Extract name
        name = None
        name_match = re.search(r'NAME\s*[:：]\s*(.+)', message_text)
        if name_match:
            name = name_match.group(1).strip()
        else:
            name_match = re.search(r'/catch\s*([^\s\n]+)', message_text)
            if name_match:
                name = name_match.group(1).strip()

        if not name:
            return

        # Clean name
        name = re.sub(r'[^\w\s\[\]\(\)\-]', '', name)
        logger.info(f"🎯 Name: {name}")

        # Send to original group
        original_chat_id = last_image_info.get('chat_id')
        if not original_chat_id:
            return

        # ===== ONLY SEND /catch (NOT the long format) =====
        await client.send_message(original_chat_id, f"/catch {name}")
        logger.info(f"⚡ AUTO CATCH: /catch {name}")

        # Send confirmation to Saved Messages
        await client.send_message(
            'me',
            f"✅ Caught: {name}\n🏷️ Group: {last_image_info.get('chat_title', 'Unknown')}"
        )

        last_image_info = {}

    except Exception as e:
        logger.error(f"❌ Error: {e}")

# ===== Command: Get all saved links =====
@client.on(events.NewMessage(pattern='/getlinks'))
async def get_all_links(event):
    try:
        if not saved_groups:
            await client.send_message('me', "❌ No groups saved yet.")
            return

        message = "📋 Saved Groups:\n\n"
        for i, group in enumerate(saved_groups, 1):
            message += f"{i}. {group['chat_title']}\n   🔗 {group['link']}\n\n"

        await client.send_message('me', message)
    except Exception as e:
        logger.error(f"❌ Error: {e}")

# ===== Command: Get specific group link =====
@client.on(events.NewMessage(pattern='/link (.*)'))
async def get_group_link_command(event):
    try:
        query = event.pattern_match.group(1).strip()

        found = None
        for group in saved_groups:
            if query.lower() in group['chat_title'].lower() or query == str(group['chat_id']):
                found = group
                break

        if found:
            await client.send_message('me', f"🏷️ {found['chat_title']}\n🔗 {found['link']}")
        else:
            await client.send_message('me', f"❌ No group found: {query}")
    except Exception as e:
        logger.error(f"❌ Error: {e}")

async def main():
    try:
        await client.start(phone=PHONE_NUMBER)
        logger.info("="*60)
        logger.info("🤖 SIMPLE AUTO CATCHER STARTED!")
        logger.info(f"📡 Monitoring: Character_Catcher_Bot images")
        logger.info(f"⚡ Auto catch: ONLY /catch <name>")
        logger.info(f"📝 Commands: /getlinks, /link <name>")
        logger.info("="*60)

        await client.run_until_disconnected()
    except Exception as e:
        logger.error(f"❌ Failed to start: {e}")

if __name__ == "__main__":
    asyncio.run(main())

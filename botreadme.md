Invite link for it:
https://discord.com/oauth2/authorize?client_id=1282690365618651299&permissions=2147609600&scope=bot%20applications.commands

Commands:
/setup (setup for your fivem server) 
/services (shows service list)

Setup command options explained:
host = database host name
user = database username
password = database password
database = database name
api_ip = ip of your fivem server
api_port = port of your fivem server



**INSTRUCTIONS TO MAKE IT REFRESH INSTANTLY**
*Go to "qb-core>server>commands" and search for "setjob" which will look something like this*


QBCore.Commands.Add('setjob', Lang:t('command.setjob.help'), { { name = Lang:t('command.setjob.params.id.name'), help = Lang:t('command.setjob.params.id.help') }, { name = Lang:t('command.setjob.params.job.name'), help = Lang:t('command.setjob.params.job.help') }, { name = Lang:t('command.setjob.params.grade.name'), help = Lang:t('command.setjob.params.grade.help') } }, true, function(source, args)
    local Player = QBCore.Functions.GetPlayer(tonumber(args[1]))
    if Player then
        Player.Functions.SetJob(tostring(args[2]), tonumber(args[3]))
    else
        TriggerClientEvent('QBCore:Notify', source, Lang:t('error.not_online'), 'error')
    end
end, 'admin')


*Replace it with this*


QBCore.Commands.Add('setjob', Lang:t('command.setjob.help'), { 
    { name = Lang:t('command.setjob.params.id.name'), help = Lang:t('command.setjob.params.id.help') }, 
    { name = Lang:t('command.setjob.params.job.name'), help = Lang:t('command.setjob.params.job.help') }, 
    { name = Lang:t('command.setjob.params.grade.name'), help = Lang:t('command.setjob.params.grade.help') } 
}, true, function(source, args)
    local Player = QBCore.Functions.GetPlayer(tonumber(args[1]))
    if Player then
        local jobName = tostring(args[2])
        local grade = tonumber(args[3])
        
        -- Fetch job details from jobs.lua
        local jobDetails = getJobDetails(jobName, grade)
        
        if jobDetails then
            -- Set the job for the player
            Player.Functions.SetJob(jobName, grade)
            
            -- Refresh the database with the job details
            refreshPlayerJobInDatabase(Player.PlayerData.citizenid, jobDetails)
            TriggerClientEvent('QBCore:Notify', source, Lang:t('success.job_set'), 'success')
        else
            TriggerClientEvent('QBCore:Notify', source, Lang:t('error.job_not_found'), 'error')
        end
    else
        TriggerClientEvent('QBCore:Notify', source, Lang:t('error.not_online'), 'error')
    end
end, 'admin')

-- Function to fetch job details from jobs.lua
function getJobDetails(jobName, grade)
    local jobData = QBCore.Shared.Jobs[jobName]  -- Accessing the jobs directly from QBCore

    if jobData then
        local jobDetails = {
            grade = {
                name = tostring(grade),
                isboss = (grade == jobData.bossGrade),  -- Assuming you have a boss grade defined
                level = tostring(grade),
                payment = jobData.payment  -- Adjust if necessary to get payment correctly
            },
            isboss = (grade == jobData.bossGrade),
            type = jobData.type,
            payment = jobData.payment,
            name = jobName,
            onduty = true,  -- Set to true upon job assignment
            label = jobData.label
        }
        return jobDetails
    end
    return nil
end

-- Function to refresh the player's job in the database
function refreshPlayerJobInDatabase(citizenid, jobDetails)
    local query = [[
        UPDATE players
        SET job = ?
        WHERE citizenid = ?
    ]]

    -- Prepare the job JSON structure
    local jobJson = json.encode(jobDetails)  -- Convert the job details to JSON
    local params = { jobJson, citizenid }

    exports.ghmattimysql:execute(query, params, function(affectedRows)
        if affectedRows then
            print("Player job updated in the database successfully.")
        else
            print("Failed to update player job in the database.")
        end
    end)
end

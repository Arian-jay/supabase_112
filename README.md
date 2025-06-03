"# supabase_112" 

copy .env.example file

#cd backend
#cp .env.example .env

--Update the .env file:

    Set the correct SUPABASE_URL
    Set the correct SUPABASE_ANON_KEY


--build and start the containers: 
    docker-compose up --build
    docker-compose up -d

--to show the running container:
    #docker ps

--commands for stopping the backend or frontend
    #docker stop <container_id_of_backend>
    #docker stop <container_id_of_frontend>

--commands for starting again the backend or frontend
    #docker start <container_id_of_backend>
    #docker start <container_id_of_frontend>

--command for turning all the container down:
    #docker-compose down



database query:

create table todos (
  id serial primary key,
  task text not null,
  is_completed boolean default false,
  created_at timestamp with time zone default current_timestamp
);